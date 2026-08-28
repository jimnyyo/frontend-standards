---
name: static-site-includes
description: Share header, footer, side navigation and other common regions across a static multi-page site that has no build tool or server-side includes, by fetching HTML fragments at runtime — the loadInc helper, the placeholder comment convention that keeps the source path visible in the markup, callback initialization for scripts that depend on injected DOM, null-guarding pages that omit a region, and marking the current menu by matching the URL instead of a hand-maintained index. Use when building or fixing shared page regions, adding a page that needs the common header/footer, wiring up a script that stops working after an include, or replacing duplicated inline headers. [한글] 빌드툴·서버사이드 인클루드 없는 정적 사이트에서 헤더·푸터·사이드메뉴·팝업을 fetch로 공통화하는 방법. "헤더 공통으로 빼줘", "푸터 따로 관리", "include", "공통 영역", "페이지마다 헤더 복사", "메뉴 활성화 안 돼", "현재 메뉴 표시", "헤더 넣었더니 JS가 안 먹어", "새 페이지 만들 때 공통" 요청에 사용.
---

# Static site common regions via fetch includes

## 요약 (Korean summary)

빌드툴도 서버사이드 인클루드도 없는 정적 사이트에서 공통 영역을 한 번만 관리하는 방법입니다.

| 절 | 핵심 내용 |
|---|---|
| 1. loadInc | `fetch` → `innerHTML` → 콜백. 30줄 안 되는 헬퍼 하나로 끝난다. |
| 2. null 가드 | 대상 엘리먼트가 없으면 **조용히 건너뛴다**. 이게 있어야 같은 스크립트를 모든 페이지에 그대로 쓴다. |
| 3. 플레이스홀더 주석 | `<div id="site-header"></div><!-- /경로/header.html -->` — 마크업만 봐도 무엇이 들어오는지 보이게. |
| 4. 콜백 초기화 | 삽입된 DOM에 붙는 JS는 **반드시 콜백 안에서** 초기화. 밖에서 부르면 아직 없는 요소를 찾는다. |
| 5. 현재 메뉴 | 수동 인덱스 대신 **URL 경로 매칭**으로 `active` 부여. |
| 6. 한계 | fetch 기반이므로 `file://` 에서 안 되고 SEO·FOUC 트레이드오프가 있다. |

구현 전문은 `references/loadinc.md`에 있습니다.

---

## 1. The helper

Everything rests on one function. Keep it this small.

```js
function loadInc(id, path, callback) {
  fetch(path)
    .then(res => res.text())
    .then(html => {
      const el = document.getElementById(id);
      if (!el) return;              /* 이 영역이 없는 페이지는 건너뜀 */
      el.innerHTML = html;
      if (callback) callback();
    });
}
```

Call sites sit at the top of the shared script, once, for the whole site:

```js
loadInc('site-header', '/<project>/inc/header.html', () => {
  initMnav(); initGnbActive(); initRelDrop();
});
loadInc('site-footer', '/<project>/inc/footer.html', () => {
  updateFloat(); initFtRelDrop();
});
loadInc('site-poll', '/<project>/inc/poll.html');
```

Fragment files contain **markup only, no `<script>`**. Scripts injected via `innerHTML` do not execute — this is a silent failure that costs an hour if you do not know it. All behavior lives in the shared script and is wired through the callback.

## 2. The null guard is the important line

`if (!el) return;` looks like defensive noise. It is the reason one script can serve every page.

Not every page has every region — the satisfaction poll is not on the login page, the notice popup is only on the index. Without the guard, every page that omits a region throws, and the usual "fix" is a per-page script or a growing list of `if (location.pathname.includes(...))` conditions. With it, **the page's markup decides**: put the placeholder div in, the region loads; leave it out, nothing happens.

So the inclusion rule becomes declarative and lives in the HTML, not in JS branching.

## 3. Placeholder comment convention

Every include target carries an inline comment naming the file it loads:

```html
<div id="site-header"></div><!-- /<project>/inc/header.html -->
<div id="site-footer"></div><!-- /<project>/inc/footer.html -->
<div id="notice-popup"></div><!-- /<project>/inc/noticePopup.html -->
```

An empty `<div id="site-header">` tells the next person nothing. With the comment, the markup is readable on its own and the file is one click away — and this matters more than usual here, because the content genuinely is not in the file you are looking at.

This applies inside fragments too: if `header.html` has its own include target, it gets the same treatment.

## 4. Callbacks — anything touching injected DOM

Code that queries elements from a fragment must run **after** that fragment lands. `fetch` is async, so any initialization at the bottom of the script file runs first and finds nothing.

```js
/* 잘못 — 헤더가 아직 없다 */
loadInc('site-header', '/<project>/inc/header.html');
initGnbActive();

/* 올바름 */
loadInc('site-header', '/<project>/inc/header.html', () => { initGnbActive(); });
```

Two cases that are easy to miss:

- **A region that lives in another region's fragment.** If the floating buttons are inside `footer.html`, whatever positions them (`updateFloat`) belongs in the *footer's* callback, not the page's load event.
- **Nested includes.** A fragment containing its own placeholder needs its `loadInc` called from the parent's callback — chained, not parallel.

Symptom when this is wrong: the region renders correctly but is dead — no toggle, no active state, no scroll behavior.

## 5. Current menu by URL, not by hand

The bad version is a per-page index (`initGnb(2)`) or a hardcoded `active` class in each copy of the header. Both defeat the point of having one header file.

Match the URL instead:

```js
function initGnbActive() {
  const cur = location.pathname.split('/').filter(Boolean)[1];   /* /<project>/<Category>/page.html → Category */
  if (!cur) return;
  document.querySelectorAll('.gnb_mn > a').forEach(a => {
    if (new URL(a.href).pathname.split('/').filter(Boolean)[1] === cur) a.classList.add('active');
  });
}
```

- `filter(Boolean)` drops the empty segment from the leading slash; index `[1]` is the category because `[0]` is the project root. **Adjust the index to the actual depth** — a site served from the domain root uses `[0]`.
- Compare `new URL(a.href).pathname`, not the raw `href` attribute, so relative and absolute links both resolve.
- Add a new page under an existing category and it highlights correctly with no edit at all.

Mobile menus and side navigation use the same approach with their own selector, matching one segment deeper when they represent sub-depth.

## 6. Paths, and what this approach costs

**Paths inside fragments must be absolute.** A fragment is fetched from one place but executes inside pages at many different depths, so a relative `img/logo.png` resolves differently per page and breaks. Use the project-rooted form (`/<project>/img/logo.png`) everywhere inside fragment markup and in any JS that references it.

Meanwhile the page's own `<link>` and `<script>` tags can stay relative, since they resolve against the page itself.

Known trade-offs — accept them deliberately or pick another approach:

| Cost | Detail |
|---|---|
| No `file://` | `fetch` fails on the filesystem protocol. Local work needs a dev server. |
| Not in the initial HTML | Crawlers that do not execute JS see empty placeholders. Fine for internal or authenticated pages; think twice for landing pages that need SEO. |
| Brief empty space | The header paints one frame late. Reserve its height in CSS so the page does not jump. |
| Extra requests | One per region. Negligible at this scale, but it is not free. |

If any of these is a blocker, the answer is server-side includes or a build step — not working around `fetch`.

## Before you add a shared region, check

1. Does the fragment contain a `<script>` tag? It will not run — move it to the shared script.
2. Is anything that touches this fragment's DOM being initialized outside the callback?
3. Does the placeholder div have its path comment?
4. Are all paths inside the fragment absolute?
5. Does the region's script assume the element exists, or is it null-guarded?
6. Is the current-menu state derived from the URL, or hand-maintained per page?
