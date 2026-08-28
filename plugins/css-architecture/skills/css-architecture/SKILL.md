---
name: css-architecture
description: How to organize CSS files and define the shared foundation for a static site with no build tool — split stylesheets by component nature rather than by page, decide what is shared versus page-only, fix the load order, name design tokens on :root, and write the reset with `:where()` so every default has specificity 0. Use when setting up a new project's CSS, adding or splitting a stylesheet, deciding where a new component's rules belong, naming or auditing CSS variables, writing or fixing a reset, or when styles are colliding across pages. Not for how to write an individual rule — that is the conventions skill. [한글] 정적 사이트(빌드툴 없음)의 CSS 파일 구성과 공통 기반. "CSS 파일 어떻게 나눠", "이 스타일 어디에 넣어", "새 프로젝트 세팅", "basic.css", "common.css", "리셋 만들어줘", "CSS 변수", "디자인 토큰", "색상 변수 정리", "로드 순서", "스타일이 겹쳐", "다른 페이지에 영향 가", "특이도" 요청에 사용. 개별 규칙 작성법이 아니라 파일 단위 구성과 토큰·리셋 정의에 적용.
---

# CSS architecture for build-tool-free static sites

## 요약 (Korean summary)

CSS를 **어디에 둘지**와 **공통 기반을 어떻게 정의할지**의 규칙입니다. 규칙 하나를 어떻게 쓰는지는 `html-layout-conventions` 스킬에 있습니다.

| 절 | 핵심 내용 |
|---|---|
| 1. File split | 파일은 **페이지가 아니라 컴포넌트 성격** 기준으로 나눈다. 게시판이면 어느 페이지에서 쓰든 `board.css`. |
| 2. 공통 vs 페이지 | 2개 이상 페이지에서 재사용되면 공통 파일, 1회성·충돌위험·무거운 1페이지 전용이면 페이지 CSS로 분리. |
| 3. Load order | `basic → common → sub 또는 main → 페이지 전용`. 빌드툴이 없으므로 **소스 순서가 곧 우선순위**. |
| 4. Design tokens | `:root`에 색·폰트크기·배경·버튼·반경·그림자를 표준 이름으로. 컴포넌트에서 raw hex 금지. |
| 5. Reset | 모든 기본값은 `:where()`로 특이도 0. `body *`(0,0,1)로 쓰면 폼 기본값을 덮어써 버그가 난다. |
| 6. Page CSS 금지사항 | 페이지 전용 CSS에 `h2`·`p` 같은 태그 글로벌 셀렉터 금지 — 반드시 클래스 기반. |

기준 리셋 전문은 `references/basic-css-baseline.md`에 있습니다.

---

## 1. Split files by component nature, not by page

The instinct is to make one stylesheet per page. It does not survive contact with a real site: the board list appears on the notice page, the data page, and the my-page application history, and three copies immediately drift apart.

**The rule: a component's rules live in the file that owns that kind of component, regardless of which page uses it.**

A typical split:

| File | Owns |
|---|---|
| `basic.css` | Reset + `:root` design tokens. Nothing else. |
| `common.css` | Header, GNB, footer, floating buttons, site-wide sections, shared motion |
| `main.css` | Index-only sections |
| `sub.css` | Sub page shell (container, sidebox, breadcrumb) + general content components (tabs, tables, form rows, accordions, buttons, layer popups) |
| `board.css` | **Every** `bd_*` list, its search bar, and all its variants — wherever they appear |
| `member.css` | Member/mypage-only UI |

The test when adding something new is not "which page am I on" but "what kind of thing is this". A board variant built for the my-page application history still goes in `board.css`, not `member.css`. An accordion used only by the member flow still goes in `sub.css` if it is the general accordion.

Get this wrong and the symptom is specific: you go to change the board row height and find you have to change it in three files.

## 2. Shared file or page-only file?

| Put it in the shared file when | Split it into a page-only file when |
|---|---|
| Two or more pages use the pattern | It is a one-off visual that will never recur |
| It is a variant of something shared | It risks colliding with shared rules |
| | It is heavy and belongs to exactly one page (a chart, an org diagram, a complex popup) |

When a one-off later turns up on a second page, move it into the shared file at that moment — do not copy it.

## 3. Load order is the cascade

With no build tool and no nesting, **source order is the only priority mechanism you have** besides specificity. Fix it and never vary it per page:

```html
<link rel="stylesheet" href="css/basic.css">    <!-- reset + tokens -->
<link rel="stylesheet" href="css/common.css">   <!-- header·footer·shared -->
<link rel="stylesheet" href="css/sub.css">      <!-- or main.css on index -->
<link rel="stylesheet" href="css/board.css">    <!-- only if this page has one -->
<link rel="stylesheet" href="css/detail.css">   <!-- page-only, last -->
```

Load only what the page needs. A page-type table keeps this decidable rather than per-page improvisation:

| Page type | Loads |
|---|---|
| Index | basic + common + main |
| Sub page | basic + common + sub |
| Sub page with a board | basic + common + sub + board |
| Page with one-off heavy UI | … + that page's CSS, last |

## 4. Design tokens on `:root`

Every color, size, and radius a component uses comes from a token. A raw hex in a component rule is a value that can never be changed site-wide.

Standard names — reuse these rather than coining parallel ones:

```css
:root {
  /* 색 */
  --color-primary: …; --color-secondary: …; --color-accent: …; --color-gray: …;

  /* 글자색 — 역할 기준 */
  --color-text-title: …; --color-text-body: …; --color-text-small: …;
  --color-text-placeholder: …; --color-text-disabled: …;

  /* 상태색 */
  --color-info: …; --color-danger: …; --color-warning: …; --color-success: …;

  /* 버튼 */
  --btn-bg: …; --btn-bg-hover: …; --btn-bg-disabled: …; --btn-border-hover: …;

  /* 배경·테두리 — 층위 기준 */
  --bg_color: …; --bg-layer1: …; --bg-layer2: …; --bg-layer3: …; --bg-layer4: …;
  --bg-hover: …; --border_color: …;

  /* 글자 크기 */
  --font14: 14px; --font15: 15px; --font16: 16px; --font17: 17px;
  --font18: 18px; --font20: 20px; --font22: 22px; --font24: 24px;
  --font26: 26px; --font30: 30px; --font32: 32px; --font36: 36px;

  /* 그 외 */
  --font_main: …; --radius: …; --shadow: …; --transition: …;
  --const-header-height: …; --const-footer-height: …;
}
```

Two things this buys that are easy to lose:

- **Text colors are named by role, not by shade.** `--color-text-small` survives a redesign that changes the actual gray; `--color-999` does not.
- **Backgrounds are named by layer.** `--bg-layer1` through `--bg-layer4` say how deep the surface is. A component asking for "the second-level background" keeps working when the palette shifts.

Legacy `--col-<hex>` style tokens (`--col-1a1a1a`) are the anti-pattern this replaces — they encode the value in the name, so renaming is the only way to change them. Do not add new ones.

### Breakpoints

Fix the breakpoint numbers in the foundation and use only those. A typical set is **1280 / 768 / 480**, with a wide tier above when a max-width layout needs one. No page invents an intermediate value; if a layout genuinely needs one, it is added to the foundation for everyone.

CSS variables do not work inside `@media` queries, so these live as a written rule rather than as tokens — which makes it more important that they are stated once and followed.

## 5. The reset — every default at specificity 0

This is the rule that causes real bugs when broken, so it gets stated precisely.

**Every declaration in the reset must be wrapped in `:where()`.**

```css
/* 권장 — 특이도 0 */
:where(body, body *, input, textarea, button, select) { margin: 0; padding: 0; box-sizing: border-box; }
:where(input[type='text'], select, textarea) { height: 48px; padding: 0 16px; border: 1px solid var(--border_color); }

/* 금지 */
body * { margin: 0; padding: 0; }                    /* (0,0,1) */
input[type='text'] { height: 48px; padding: 0 16px; } /* (0,1,1) — 클래스보다 높다 */
```

Two failure modes, both of which have actually happened:

- **`input[type='text']` is (0,1,1)**, higher than any single class (0,1,0). A component writing `.srch_inp { border: none; }` loses to the reset, and the developer adds `!important` to a base style — which then has to be fought everywhere.
- **A bare `body *` is (0,0,1)**, which beats a `:where()` rule at (0,0,0). When the global reset is bare and the form defaults are wrapped, `body *`'s `padding: 0` wins over the form's `padding: 0 16px`, and input text sits flush against the left edge. The fix is not raising the form's specificity — it is lowering the reset's.

The invariant: **all defaults are specificity 0; control comes from source order and classes only.**

See `references/basic-css-baseline.md` for a complete reset that satisfies this, including the form, radio, and checkbox defaults components are expected not to redeclare.

## 6. Rules for page-only stylesheets

A page-only file is loaded last, so anything global in it silently overrides the whole site.

- **No bare tag selectors.** `h2 { font-size: 32px; }` in `detail.css` restyles every `h2` on every page that also loads it — and eventually one will. Always class-based.
- **No token redefinition.** A page that needs a different primary color needs a modifier class, not a local `--color-primary`.
- Scope generously: prefix rules with the page's own root class when the file is large.

## Before you add a stylesheet, check

1. Is this actually a new *kind* of component, or a variant of one that already has a home?
2. Will a second page use this? If yes, it belongs in a shared file now, not later.
3. Is the load order still `basic → common → shared → page-only`?
4. Any raw hex or px in a component rule that should be a token?
5. Any reset or default declaration not wrapped in `:where()`?
6. Any bare tag selector in a page-only file?
