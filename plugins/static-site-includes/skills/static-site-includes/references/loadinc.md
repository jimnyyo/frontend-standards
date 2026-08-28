# Reference implementation — fetch includes

Complete working code for the pattern. Drop into the shared script (`common.js`), adjust `/<project>/` to the real root.

---

## 1. The helper and its call sites

Put these at the **top** of the shared script, before the function definitions they reference — the fetches are async, so the functions exist by the time the callbacks run.

```js
/* ============================================================
   공통 영역 비동기 삽입
   - 대상 엘리먼트가 없는 페이지는 조용히 건너뜀 (null 가드)
   - 삽입된 DOM에 붙는 초기화는 반드시 callback 에서
   ============================================================ */
function loadInc(id, path, callback) {
  fetch(path)
    .then(res => res.text())
    .then(html => {
      const el = document.getElementById(id);
      if (!el) return;
      el.innerHTML = html;
      if (callback) callback();
    });
}

/* 헤더 — 전체메뉴·GNB 활성화·연관사이트 드롭다운 초기화 */
loadInc('site-header', '/<project>/inc/header.html', () => {
  initMnav();
  initGnbActive();
  initMnavActive();
  initRelDrop();
});

/* 푸터 — .float 버튼이 푸터 안에 있으므로 위치 계산도 여기서 */
loadInc('site-footer', '/<project>/inc/footer.html', () => {
  updateFloat();
  initFtRelDrop();
});

/* 검색 레이어 — 별도 파일, 삽입 후 패널 동작 초기화 */
loadInc('site-search', '/<project>/Common/search.html', initSearch);

/* 만족도 조사 — #site-poll 플레이스홀더가 있는 페이지에만 자동 삽입 */
loadInc('site-poll', '/<project>/inc/poll.html');
```

Note the last one: no callback, no condition. Pages that want the poll add the placeholder; pages that do not, omit it. The null guard does the rest.

---

## 2. Page markup

```html
<body>
  <div id="site-header"></div><!-- /<project>/inc/header.html -->

  <div class="container">
    <aside class="sidebox">
      <div id="side-nav"></div><!-- /<project>/inc/side_intro.html -->
    </aside>
    <div class="content">
      <!-- 페이지 본문 -->
      <div id="site-poll"></div><!-- /<project>/inc/poll.html -->
    </div>
  </div>

  <div id="site-footer"></div><!-- /<project>/inc/footer.html -->
  <div id="notice-popup"></div><!-- /<project>/inc/noticePopup.html -->

  <script src="js/common.js"></script>
</body>
```

Reserve the header height in CSS so the page does not jump when the fragment lands:

```css
#site-header { min-height: var(--const-header-height); }
```

---

## 3. Fragment file rules

`inc/header.html` — **markup only.**

```html
<div class="gov">…</div>

<header class="header">
  <a href="/<project>/" class="logo"><img src="/<project>/img/logo.png" alt="로고"></a>

  <nav class="gnb">
    <div class="gnb_mn"><a href="/<project>/Intro/about.html">재단소개</a></div>
    <div class="gnb_mn"><a href="/<project>/Biz/personal.html">사업소개</a></div>
  </nav>

  <div id="site-search"></div><!-- /<project>/Common/search.html -->
</header>

<div class="mnav_dim"></div>
<div id="mnav">…</div>
```

Three things this shows:

- Every path is **absolute** — the fragment runs inside pages at several directory depths.
- **No `<script>` tag.** Scripts inserted through `innerHTML` never execute.
- A nested include target gets its own path comment, same as in a page.

---

## 4. Current-menu matching

```js
/* GNB — /<project>/<Category>/page.html 의 <Category> 로 매칭
   ※ filter(Boolean) 이 선행 슬래시의 빈 조각을 제거하므로
     [0] = 프로젝트 루트, [1] = 카테고리.
     도메인 루트에서 서비스하는 사이트라면 [0] 을 쓸 것. */
function initGnbActive() {
  const cur = location.pathname.split('/').filter(Boolean)[1];
  if (!cur) return;
  document.querySelectorAll('.gnb_mn > a').forEach(a => {
    const link = new URL(a.href).pathname.split('/').filter(Boolean)[1];
    if (link === cur) a.classList.add('active');
  });
}

/* 모바일 메뉴 — 같은 방식, 선택자만 다름 */
function initMnavActive() {
  const cur = location.pathname.split('/').filter(Boolean)[1];
  if (!cur) return;
  document.querySelectorAll('#mnav a[href]').forEach(a => {
    const link = new URL(a.href).pathname.split('/').filter(Boolean)[1];
    if (link === cur) a.classList.add('active');
  });
}

/* 사이드 메뉴 — 파일명까지 비교해 한 뎁스 더 정확하게 */
function initSideActive() {
  const cur = location.pathname.split('/').filter(Boolean).pop();
  if (!cur) return;
  document.querySelectorAll('.side_nav a').forEach(a => {
    if (new URL(a.href).pathname.split('/').filter(Boolean).pop() === cur) {
      a.classList.add('active');
      a.closest('.side_dep')?.classList.add('open');   /* 2뎁스면 부모도 펼침 */
    }
  });
}
```

Compare `new URL(a.href).pathname` rather than the raw attribute so relative and absolute hrefs both normalize.

---

## 5. Chained (nested) includes

When a fragment contains another placeholder, load it **from the parent's callback**. Calling both at the top level is a race — the child's target may not exist yet.

```js
loadInc('site-header', '/<project>/inc/header.html', () => {
  initMnav(); initGnbActive();
  loadInc('site-search', '/<project>/Common/search.html', initSearch);   /* 헤더 안에 있음 */
});
```

---

## 6. Callbacks that depend on injected geometry

Anything measuring position must run after the fragment renders, and usually again on resize/scroll.

```js
/* 푸터가 뷰포트에 들어오면 플로팅 버튼을 푸터 위로 밀어 올림 */
function updateFloat() {
  const float = document.querySelector('.float');
  const footer = document.querySelector('.footer');
  if (!float || !footer) return;                      /* 여기도 null 가드 */

  const onScroll = () => {
    const top = footer.getBoundingClientRect().top;
    const vh = window.innerHeight;
    float.style.bottom = top < vh ? (vh - top + 100) + 'px' : '';   /* '' → CSS 기본값 복귀 */
  };
  onScroll();
  window.addEventListener('scroll', onScroll, { passive: true });
  window.addEventListener('resize', onScroll);
}
```

`.float` lives inside `footer.html`, so this is called from the footer's callback — not from `DOMContentLoaded`.

---

## 7. Local development

`fetch` fails under `file://`. Serve the folder over HTTP:

```
VS Code Live Server  → 프로젝트 상위 폴더를 루트로 설정
  http://127.0.0.1:5500/<project>/index.html

또는
  npx serve .
  python -m http.server 5500
```

Set the server root **above** the project folder so that `/<project>/…` absolute paths resolve exactly as they will in production.
