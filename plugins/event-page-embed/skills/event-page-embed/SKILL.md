---
name: event-page-embed
description: Building a one-off event, campaign, or landing page that gets pasted whole into a CMS body field on a site you do not control — a single self-contained file, every class and rule scoped under one page prefix so nothing leaks either way, winning the specificity fight against the host template's own rules, splitting PC and mobile when both share one body field, subsetting fonts, and keeping dated revision files across review rounds. Use when a page will be delivered as a block to paste into an editor rather than as files on a server, and whenever an embedded page misbehaves only on one device or only inside the CMS. This replaces the multi-file conventions — do not apply the css-architecture or static-site-includes skills to a page like this. [한글] 남의 CMS 본문 필드에 통째로 붙여 넣는 이벤트·캠페인·랜딩 상세 페이지. "이벤트 페이지", "캠페인 페이지", "상세페이지 만들어줘", "랜딩", "에디터에 붙일 거", "본문에 넣을 코드", "한 파일로", "CMS에 넣으면 깨져", "PC만 깨져", "모바일만 멀쩡해", "원래 사이트 스타일이랑 충돌", "헤더가 두 개 나와" 요청에 사용. 파일을 서버에 올리는 게 아니라 블록 하나를 전달하는 작업일 때 적용. 이 경우 `css-architecture`·`static-site-includes` 규칙은 적용하지 않는다.
---

# Event pages embedded in someone else's CMS

## 요약 (Korean summary)

파일을 서버에 올리는 게 아니라 **에디터 본문에 붙여 넣는** 페이지의 규칙입니다. 다른 스킬(`css-architecture`·`static-site-includes`)과 전제가 반대이므로 섞지 마세요.

| 절 | 핵심 내용 |
|---|---|
| 1. 전제가 다름 | 파일 분할·공통 CSS·include가 전부 불가능. 단일 파일 안에 CSS·JS를 다 넣는다. |
| 2. 삽입 블록 | `.html` 파일은 미리보기용 껍데기. 실제 납품물은 마커 사이. 경계를 주석으로 명시. |
| 3. 스코프 | 페이지 고유 접두어 하나로 래퍼·클래스·CSS를 전부 감싼다. 변수도 `:root` 아님. |
| 4. 특이도 | 호스트 템플릿이 래퍼에 거는 규칙(주로 `img`)을 이겨야 한다. 접두어를 겹쳐 올린다. |
| 5. PC/MO | 본문 필드를 공유하면 코드를 나눌 수 없다. 스타일시트는 `media='not all'`로 끄고, `<picture>`는 1×1 GIF로 무력화. |
| 6. 폰트 | 쓰는 글자만 서브셋. 원본 CDN을 `src` 폴백으로. 상단 폰트만 preload. |
| 7. 호스트 존중 | 전역 콜백을 덮지 말 것. JS 없이도 내용은 보여야 한다. |
| 8. 리비전 | 수정 라운드마다 날짜 파일명으로 남긴다. |

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

The one thing that carries over is the naming discipline — underscore, lowercase, `prefix_meaning` — because it costs nothing and keeps the block readable.

## 2. The file is a harness; the block is the deliverable

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

## 3. Scope everything under one prefix

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

### Hiding the host's chrome

When the page brings its own header or footer, hide the host's without hiding your own:

```css
header:not(.hw26_header) { display: none !important; }
footer:not(.hw26_footer) { display: none !important; }
```

This is the one place a selector may leave the wrapper, because it has to reach the host's markup. Use `:not()` so your own is exempt, and confirm the host actually uses `<header>` rather than `<div class="head">` before relying on it.

## 4. Winning the specificity fight

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

## 5. PC and mobile when the body field is shared

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

## 6. Fonts

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

## 7. Do not disturb the host

Your block runs inside someone else's page, and the failure mode is breaking *their* code, which nobody will attribute to you.

- **Never overwrite a global callback — chain it.** `onYouTubeIframeAPIReady` and similar are single global names the host may already use. Keep a reference to the existing one and call it after yours.
- **Reach the wrapper via `document.currentScript.parentNode`**, not `document.querySelector`. If the block ever appears twice on a page, each copy then wires up its own DOM.
- **Set a JS-enabled flag rather than hiding content by default.** `root.className += ' <prefix>_js'` right inside the wrapper, then gate animations on that class. With JS blocked the content still shows — the same reason scroll-reveal's default-invisible state is dangerous.
- **Scope every `querySelector` to your wrapper.** A document-wide query will find the host's elements.

## 8. Revisions

Review rounds arrive as documents with dates on them. Mirror those dates in the filenames:

```
index.html            ← 현재 작업본
index_260820.html     ← 8/20 수정의견 반영본
index_260828.html     ← 8/28 수정의견 반영본
```

Since these pages live in a CMS rather than in version control, the file set *is* the history — it is how you answer "what did we send on the 20th". Keep the working file named plainly and snapshot on each delivery.

## Before you hand the block over

1. Does every selector start with the page prefix? (Only the host-chrome `:not()` rules may not.)
2. Are the variables on the wrapper rather than `:root`?
3. Are `<style>`, `<script>`, and any `<link>` all inside the wrapper?
4. Are the insertion markers present and accurate?
5. Was the host's own rule on your block actually inspected, or assumed?
6. Does the host force a `min-width` that makes a browser resize a false mobile test?
7. If PC and mobile share the body field, is each device-specific piece disabled — stylesheets by `media`, `<picture>` by placeholder?
8. Does the font subset cover the current copy, and does its fallback `src` still resolve?
9. Does the page still show its content with JS disabled?
10. Is any global the host might own being overwritten instead of chained?
