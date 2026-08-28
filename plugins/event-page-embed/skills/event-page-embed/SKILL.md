---
name: event-page-embed
description: Building a one-off event, campaign, or landing page that gets pasted whole into a CMS body field on a site you do not control — a single self-contained file, every class and rule scoped under one page prefix so nothing leaks either way, winning the specificity fight against the host template's own rules, splitting PC and mobile when both share one body field, subsetting fonts, and keeping dated revision files across review rounds. It also covers the two ways such a page meets the host's own header and footer: keeping them and living inside the host's container, or hiding them and supplying its own — a decision that has to be made before any layout, since full-bleed belongs to one mode and breaks the other. Use when a page will be delivered as a block to paste into an editor rather than as files on a server, and whenever an embedded page misbehaves only on one device or only inside the CMS. The file-level skills do not apply here — do not use css-architecture or static-site-includes for a page like this — but html-layout-conventions still does, with its "never redeclare what the reset handles" rule inverted, since an embedded block has no reset of its own. [한글] 남의 CMS 본문 필드에 통째로 붙여 넣는 이벤트·캠페인·랜딩 상세 페이지. "이벤트 페이지", "캠페인 페이지", "상세페이지 만들어줘", "랜딩", "에디터에 붙일 거", "본문에 넣을 코드", "한 파일로", "CMS에 넣으면 깨져", "PC만 깨져", "모바일만 멀쩡해", "원래 사이트 스타일이랑 충돌", "헤더가 두 개 나와", "헤더 푸터 살릴까 뺄까", "사이트 헤더 안 보이게" 요청에 사용. 파일을 서버에 올리는 게 아니라 블록 하나를 전달하는 작업일 때 적용. `css-architecture`·`static-site-includes`는 적용하지 않지만, `html-layout-conventions`는 그대로 적용한다 — 단 "리셋이 하는 선언은 다시 쓰지 마라"는 반대로 뒤집힌다. **이 규칙은 신규 페이지용이다.** 이미 라이브인 페이지를 열었을 땐 요청받은 수정만 하고, 그 페이지가 쓰던 방식을 따른다 — 구조·클래스명을 규칙에 맞춰 바꾸자고 하지 않는다.
---

# Event pages embedded in someone else's CMS

## 요약 (Korean summary)

파일을 서버에 올리는 게 아니라 **에디터 본문에 붙여 넣는** 페이지의 규칙입니다. 파일 단위 스킬(`css-architecture`·`static-site-includes`)은 전제가 반대라 적용하지 않습니다. 다만 `html-layout-conventions`(네이밍·태그 깊이·CSS 작성)는 **그대로 적용**됩니다 — 리셋 규칙 한 항목만 뒤집힙니다.

| 절 | 핵심 내용 |
|---|---|
| 1. 전제가 다름 | 파일 분할·공통 CSS·include가 전부 불가능. 단일 파일 안에 CSS·JS를 다 넣는다. |
| 2. 규약 적용 범위 | 네이밍·약어·상태·태그 깊이·한 줄 선언은 그대로. 페이지 골격은 적용 안 함. **"리셋이 하는 선언은 다시 쓰지 마라"는 정반대** — 내 리셋이 없으므로 직접 써야 한다. **신규 페이지용 규칙** — 라이브 페이지는 요청받은 수정만. |
| 3. 삽입 블록 | `.html` 파일은 미리보기용 껍데기. 실제 납품물은 마커 사이. 경계를 주석으로 명시. |
| 4. 스코프 | 페이지 고유 접두어 하나로 래퍼·클래스·CSS를 전부 감싼다. 변수도 `:root` 아님. |
| 5. 호스트 chrome | **유지할지 대체할지 레이아웃 짜기 전에 정한다.** 유지면 full-bleed 금지·호스트 폭 준수, 대체면 `:not()`으로 숨기고 full-bleed 필요. |
| 6. 특이도 | 호스트 템플릿이 래퍼에 거는 규칙(주로 `img`)을 이겨야 한다. 접두어를 겹쳐 올린다. |
| 7. PC/MO | 본문 필드를 공유하면 코드를 나눌 수 없다. 스타일시트는 `media='not all'`로 끄고, `<picture>`는 1×1 GIF로 무력화. |
| 8. 폰트 | 쓰는 글자만 서브셋. 원본 CDN을 `src` 폴백으로. 상단 폰트만 preload. |
| 9. 호스트 존중 | 전역 콜백을 덮지 말 것. JS 없이도 내용은 보여야 한다. |
| 10. 리비전 | 수정 라운드마다 날짜 파일명으로 남긴다. |

호스트 CMS별 실측값은 `references/cbs-cms.md` 형식으로 정리해 둔다.

---

## 1. Why the other rules do not apply here

The multi-file conventions assume you own the server: a reset file, shared stylesheets, fetch includes, absolute paths. **None of that exists here.** You are handing over one block of HTML that gets pasted into a textarea, and everything it needs must travel inside it.

So for this kind of page:

| Normally | Here |
|---|---|
| `basic.css` reset, shared tokens | No reset at all. You inherit the host's, whatever it is. |
| Split by component into files | One file. `<style>` and `<script>` inline. |
| Header/footer via includes | The host already renders its own — your job is to not fight it. |
| `:root` variables | Variables on **your** wrapper, not `:root`. |

## 2. What carries over from the conventions, and what inverts

`html-layout-conventions` **still applies** — this skill does not replace it. Only the file-level skills (`css-architecture`, `static-site-includes`) are out.

| Rule | Here |
|---|---|
| `_` 구분자, 소문자, `접두어_의미` | 그대로. 접두어가 페이지 접두어와 겹치는 것뿐 (`hw26_hero_tt`) |
| 약어 사전 (`tt` 제목, `bd` 본문 …) | 그대로 |
| 상태 클래스 `active`·`open`·`in` | 그대로 |
| 태그 깊이 최소화, 가상 요소로 장식 | 그대로 |
| CSS 한 줄 선언 | 그대로 |
| `background` 축약과 `:hover` 예외 | 그대로 |
| absolute 자식 자리를 부모 `padding`으로 예약 | 그대로 |
| 페이지 골격 `.wrap > .header + .gnb + .container` | **적용 안 함** — 사이트 골격이지 한 장짜리 페이지 골격이 아니다 |
| **"리셋이 이미 하는 선언은 다시 쓰지 마라"** | **정반대** — 아래 참조 |

### 리셋 규칙은 뒤집힌다

The conventions say never to re-declare `list-style: none`, `cursor: pointer` on a button, or a bare `margin: 0`, because `basic.css` already does them. **Here there is no `basic.css`.** The host's reset is unknown, may reset nothing, or may reset things you did not expect.

So inside an embedded block you must declare them yourself — scoped to the wrapper:

```css
.<prefix>, .<prefix> *, .<prefix> *::before, .<prefix> *::after { box-sizing: border-box; }
.<prefix> ul, .<prefix> ol, .<prefix> li { list-style: none; margin: 0; padding: 0; }
.<prefix> h1, .<prefix> h2, .<prefix> h3, .<prefix> h4 { margin: 0; padding: 0; }
.<prefix> img { max-width: 100%; display: block; }
.<prefix> a { color: inherit; text-decoration: none; }
.<prefix> button { border: 0; background: none; cursor: pointer; }
```

Keep it to what the page actually depends on rather than porting a whole reset — every rule here is one more thing that can collide. And note that it is a **scoped** reset: a bare `ul { list-style: none }` would restyle the host's navigation.

### These rules are for new pages

**Never retrofit a page that is already live.** Everything here — the scoped reset, the prefix scheme, the state class names, the chrome mode — applies to pages being built from now on.

An existing page opened for a small fix gets **that fix only**, written in whatever style the page already uses. A live campaign page is running in a CMS where a class rename means re-pasting the whole block and re-verifying it on both templates, for no visible gain. If a page's existing approach conflicts with a rule here, follow the page and say so rather than mixing two styles into one file.

Concretely: a page that shipped with `in-view` keeps `in-view`. New pages use `in`, matching the conventions.

## 3. The file is a harness; the block is the deliverable

Keep a full `.html` file so the page can be previewed locally, but mark exactly what gets pasted:

```html
<body>
<!-- ═══════════════════════════════════════════════════════════
     여기부터 아래 '삽입 끝' 까지가 에디터에 붙여넣을 블록입니다.
     모든 클래스에 hw26_ 접두어가 붙어 있고, 스타일·스크립트 모두
     .hw26 안에만 적용되므로 상위 페이지에 영향을 주지 않습니다.
     ═══════════════════════════════════════════════════════════ -->
<div class="hw26">
  <style> … </style>
  … 내용 …
  <script> … </script>
</div>
<!-- ══════════════════ 삽입 끝 ══════════════════ -->
</body>
```

Without the markers, the next person pastes the `<!doctype>` and `<head>` too, and the CMS either strips them silently or renders a nested document. State the boundary in the file rather than in a separate handover note that will get lost.

Everything — `<style>`, `<script>`, even `<link rel="preload">` — goes **inside** the wrapper. Anything outside it will not survive the paste.

## 4. Scope everything under one prefix

Pick one short prefix per page (`hw26`, `bwexpo2026`) and use it three ways:

```html
<div class="hw26">              <!-- 래퍼 -->
  <div class="hw26_hero"> …     <!-- 클래스 접두어 -->
```

```css
.hw26 { --ink:#1c1d1d; --gold:#b7282c; --max:1700px; }   /* :root 아님 */
.hw26, .hw26 *, .hw26 *::before, .hw26 *::after { box-sizing: border-box; }
.hw26 img { max-width: 100%; display: block; }
.hw26 a { color: inherit; text-decoration: none; }
```

Three rules make this work:

- **Every selector starts with the wrapper.** A bare `.hero` or `h2` restyles the host site. There is no exception.
- **Variables live on the wrapper, not `:root`.** On `:root` they leak into the host and can collide with its own token names.
- **Re-declare the handful of things you depend on** — `box-sizing`, `img { display:block }`, `a { text-decoration:none }` — scoped to your wrapper. You do not know what the host's reset does, so assume it does nothing useful.

## 5. Host chrome — keep it or replace it

Decide this **before writing any layout**, because almost everything downstream depends on it. There are two modes, and mixing them is what produces a page whose width does not line up with the header above it.

| | A. 호스트 chrome 유지 | B. 전체 대체 |
|---|---|---|
| 호스트 header/footer | 그대로 둔다 | 전부 숨긴다 |
| 자체 chrome | 없음. 자체 푸터를 둔다면 그건 본문의 일부(CTA·안내)지 사이트 푸터가 아니다 | 직접 제공한다 |
| 가로 폭 | 호스트 컨테이너 폭을 따른다 | 화면 전체를 쓴다 |
| full-bleed | **쓰지 않는다** | 필요하다 |
| 어울리는 것 | 기존 사이트에 얹는 캠페인 본문 | 독립된 특별전시·랜딩 |

### A. 유지할 때

The block lives inside the host's container, so it must behave like host content:

- **Do not use full-bleed.** A `100vw` section will be wider than the header above it, and the mismatch reads as a bug.
- **Work within the host's container width.** Read what it actually is; do not assume.
- **If a host part gets in the way, hide that part only** — not the whole chrome:

```css
/* 호스트의 '맨 위로' 버튼과 앱 배너만 — 이 페이지에선 콘텐츠와 겹친다 */
.box.to-up > a { display: none; }
#m-footer .app-cont { display: none; }
```

Always comment why. These selectors reach outside your wrapper into markup you do not own, so the next person needs to know they were deliberate and what they were for.

### B. 대체할 때

```css
header:not(.<prefix>_header) { display: none !important; }
footer:not(.<prefix>_footer) { display: none !important; }
```

- `:not()` exempts your own, so write your header and footer with the page prefix.
- **Confirm the host actually uses `<header>`/`<footer>` elements.** If its chrome is `<div class="head">`, this rule hides nothing — inspect before relying on it.
- **You now need full-bleed** to escape the host's container:

```css
.<prefix>_full { width: 100vw; margin-left: calc(50% - 50vw); }
/* 또는 배경만 넓힐 때 */
.<prefix>_sec::before { content: ''; position: absolute; left: 50%; width: 100vw; transform: translateX(-50%); z-index: -1; }
```

- **The host's `min-width` is still in force.** Hiding the header does not remove a `main { min-width: 1200px }`, so `100vw` and the viewport can still disagree. See §6.

Both modes share one rule: these are the **only** selectors allowed to leave the wrapper. Everything else stays scoped.

## 6. Winning the specificity fight

The host template wraps your block in its own markup and styles it. You cannot delete that wrapper — the template prints it. So you have to out-specify it.

The recurring case is images:

```css
/* 호스트가 거는 규칙 — 래퍼가 템플릿 소유라 못 지운다 */
.company-box02 .max-cont img { width: 100%; height: auto; }   /* (0,2,1) */

/* 이기려면 접두어를 겹쳐 (0,3,x) 로 올린다 */
.hw26 .hw26_wrap img { width: auto; }                          /* (0,3,1) */
```

- **Measure the host's rule before writing yours.** Open the live page, inspect your block, and read what is actually winning. Guessing produces `!important` wars.
- **Raise specificity by stacking your own prefixes**, not by adding `!important`. `.hw26 .hw26_wrap` is honest and still scoped; `!important` wins once and then blocks your own later overrides.
- **The host may only style one device.** A rule that lives in the desktop template does not exist on mobile, which is why "only PC is broken" is the usual shape of this bug.

Layout rules bite the same way. A host `main { min-width: 1200px }` means the container never narrows, while your `@media` queries still fire on viewport width — so a 640px window renders the mobile layout inside a 1200px box. Check for a host `min-width` before trusting a browser resize as a mobile test.

## 7. PC and mobile when the body field is shared

First establish how the host splits devices:

1. **Does the template switch on URL or on User-Agent?** If it is UA, you cannot give PC and mobile different addresses.
2. **Is the body field shared between the two?** If yes, the *same bytes* are served to both, and there is no place to put device-specific code.

When both are true, one block has to serve both. Two techniques:

**Stylesheets — disable the whole block.** Put the mobile CSS in its own `<style>` and turn it off where it should not apply:

```html
<style id="hw26_mo"> /* 모바일 전용 규칙 전부 */ </style>
<script>
  var root = document.currentScript.parentNode;
  var mo = root.querySelector('#hw26_mo');
  if (isPcTemplate()) { root.className += ' hw26_pconly'; if (mo) mo.media = 'not all'; }
</script>
```

`media = 'not all'` deactivates the sheet entirely, which `@media` cannot do — `@media` only asks about the viewport, and the viewport lies when the host forces a minimum width.

**`<picture>` — cannot be disabled, so neutralize it.** A `<source media>` is evaluated by the browser regardless of any stylesheet, so the CSS trick does not reach it. Point the unwanted source at a 1×1 transparent GIF instead:

```html
<picture>
  <source media="(max-width: 640px)" srcset="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7">
  <img src="…/hero_pc.jpg" alt="">
</picture>
```

Or rewrite the `media` attribute from JS the same way the stylesheet is switched. Either is fine; picking one and using it consistently is what matters.

## 8. Fonts

A campaign page usually wants a display face that is far too heavy to ship whole.

- **Subset to the glyphs the page actually uses.** A 1.87MB original becomes ~60KB when it only carries the characters in your copy.
- **Keep the original CDN as a second `src`.** If the subset 404s — not uploaded yet, wrong path — the browser falls back instead of rendering nothing.
- **Host with permissive CORS.** The font is fetched from the host's domain, so `Access-Control-Allow-Origin` must allow it. Verify before delivery, not after.
- **Preload only the faces used above the fold**, not all nine weights.
- **Editing the copy invalidates the subset.** Write that in a comment next to the `@font-face` block, because the person who changes one headline six weeks later will not know.

```css
@font-face {
  font-family: 'Display';
  src: url('https://<font-host>/<page>_tt_b.woff2') format('woff2'),
       url('https://<원본 CDN>/Original-Bold.woff') format('woff');   /* 서브셋 404 시 폴백 */
  font-weight: 700; font-display: swap;
}
```

## 9. Do not disturb the host

Your block runs inside someone else's page, and the failure mode is breaking *their* code, which nobody will attribute to you.

- **Never overwrite a global callback — chain it.** `onYouTubeIframeAPIReady` and similar are single global names the host may already use. Keep a reference to the existing one and call it after yours.
- **Reach the wrapper via `document.currentScript.parentNode`**, not `document.querySelector`. If the block ever appears twice on a page, each copy then wires up its own DOM.
- **Set a JS-enabled flag rather than hiding content by default.** `root.className += ' <prefix>_js'` right inside the wrapper, then gate animations on that class. With JS blocked the content still shows — the same reason scroll-reveal's default-invisible state is dangerous.
- **Scope every `querySelector` to your wrapper.** A document-wide query will find the host's elements.

## 10. Revisions

Review rounds arrive as documents with dates on them. Mirror those dates in the filenames:

```
index.html            ← 현재 작업본
index_260820.html     ← 8/20 수정의견 반영본
index_260828.html     ← 8/28 수정의견 반영본
```

Since these pages live in a CMS rather than in version control, the file set *is* the history — it is how you answer "what did we send on the 20th". Keep the working file named plainly and snapshot on each delivery.

## Before you hand the block over

1. Does every selector start with the page prefix? (Only the §5 host-chrome rules may not.)
2. Was the chrome mode decided before layout — and does the layout match it? (유지 모드에 full-bleed가 섞이지 않았는지)
3. Are the reset declarations the page depends on written in, scoped to the wrapper?
4. Are the variables on the wrapper rather than `:root`?
5. Are `<style>`, `<script>`, and any `<link>` all inside the wrapper?
6. Are the insertion markers present and accurate?
7. Was the host's own rule on your block actually inspected, or assumed?
8. Does the host force a `min-width` that makes a browser resize a false mobile test?
9. If PC and mobile share the body field, is each device-specific piece disabled — stylesheets by `media`, `<picture>` by placeholder?
10. Does the font subset cover the current copy, and does its fallback `src` still resolve?
11. Does the page still show its content with JS disabled?
12. Is any global the host might own being overwritten instead of chained?
