---
name: html-layout-conventions
description: House rules for hand-written HTML markup and CSS — the .wrap/.header/.gnb/.container/.footer page skeleton, underscore `prefix_meaning` class naming with a shared abbreviation dictionary, modifier/state/utility class conventions, minimal tag depth, one-line CSS declaration blocks, `:where()` specificity control, and no re-declaring what the reset already handles. Use whenever markup or a stylesheet is actually being written or edited: a page, sub page, board list, form, card grid, header/footer/GNB, popup, responsive rules — including reviewing, refactoring, or renaming classes in existing code — even if the user never says "conventions" or "naming". For deciding what to build and in what order before any code exists, that is the workflow skill instead. [한글] HTML·CSS를 실제로 작성·수정할 때의 사내 규약(레이아웃 골격, 언더스코어 `접두어_의미` 네이밍과 공용 약어 사전, 모디파이어·상태·유틸 클래스, 태그 뎁스 최소화, CSS 한 줄 선언, `:where()` 특이도 제어, 리셋 중복 선언 금지). "퍼블리싱", "마크업 짜줘", "페이지·서브페이지 만들어줘", "게시판 목록", "신청 폼", "카드 리스트", "헤더/푸터/GNB", "팝업", "반응형", "CSS 정리해줘", "클래스명 바꿔줘", "코딩 컨벤션 맞춰줘", "리팩터링" 요청에 사용. 결과물이 HTML 마크업이나 스타일시트라면 기본 적용.
---

# HTML & CSS House Conventions

## 요약 (Korean summary)

우리 프로젝트 HTML·CSS 작성 규약입니다. 아래 영문 본문이 실제 규칙이고, 이 요약은 무엇이 들어 있는지 빠르게 확인하는 용도입니다.

| 절 | 핵심 내용 |
|---|---|
| 1. Page skeleton | `.wrap > .header + .gnb + (.container > .content + .sidebox) + .footer`. 1단 페이지는 `.container`·`.sidebox` 생략, 2단일 때만 추가. 이 이름들은 고정. |
| 2. Class naming | 구분자 `_`만 사용(`-` 금지), 전부 소문자, `접두어_의미` 2단어 기본. 같은 의미 = 같은 약어(제목은 어디서나 `tt`, 본문은 `bd`). 사전에 없는 약어를 새로 만들면 그 사실을 알려줄 것. |
| 3. Modifiers · states · utilities | 변형은 인덱스형(`bd_st1`)·의미형(`btn_pri`)·조합형(`.bd_list.bd_st1`), 상태는 접두어 없는 공용 단독 클래스(`active`·`open`·`in`·`done`·`err`), 유틸은 `m_b20`·`txt_center`·`f_between`. |
| 4. HTML authoring | 태그 뎁스 최소화(`grid-template-areas`, `nav > a`, `a.card`, `clip-path`), 장식·순번은 가상 요소와 `counter()`로, 이미지+텍스트는 `<a>` 하나로 묶기, 목록은 `ul`/`ol`/`li`, 표 형태 UI는 모바일에서 리스트로 전환. |
| 5. CSS authoring | 선언 블록은 한 줄, `background` 축약(단 hover 이미지 교체 시 분리), 자식 3개 이상 반복 시 `부모 > *` 기본값, 미디어쿼리 안에서도 부모 선택자로 스코핑, 모든 기본값은 `:where()`로 특이도 0, 리셋이 이미 하는 선언은 다시 쓰지 않기, absolute 자식 영역은 부모 `padding`으로 예약. |
| 6. Images | 아이콘은 `background-image`(`<img>` 아님), hover 교체는 `background-image`만, `:hover`에 `background` 축약 금지, 이미지 경로는 프로젝트 절대경로. |
| 마무리 체크 | 인계 전 9항목 체크리스트로 자주 나오는 실수를 확인. |

세부 컴포넌트 마크업은 `references/patterns.md`를 참고합니다. CSS 파일을 어떻게 나누고 리셋·디자인 토큰을 어떻게 정의하는지는 별도 `css-architecture` 스킬에 있습니다.

Markup written by different people at different times has to look like one person wrote it, because whoever maintains it next reads the class name to know where they are. Every rule below exists to make markup predictable at a glance: same meaning → same abbreviation, same page → same skeleton, same declaration → written only once.

Apply these rules by default when producing HTML or CSS. When existing code in the project contradicts a rule, follow the existing code locally and mention the discrepancy rather than silently mixing two styles inside one file.

## 1. Page skeleton

```
.wrap
  ├── header.header
  ├── nav.gnb
  ├── div.container      ← only when a 2-column layout is needed
  │     ├── div.content
  │     └── div.sidebox
  └── footer.footer
```

- One-column pages (index and similar) omit `.container` and `.sidebox` — content sections sit directly under `.wrap`. Wrapping a single column in a container that has nothing to contain is pure tag depth.
- Sub pages and anything with a side navigation add `.container` holding `.content` + `.sidebox`.
- These five names (`wrap`, `header`, `gnb`, `container`, `content`, `sidebox`, `footer`) are fixed. Do not invent `page_wrap`, `main_container`, `aside`, etc.

## 2. Class naming

- Separator is `_`. Never `-`. All lowercase.
- Components are `prefix_meaning` — two words by default. The prefix is a 2–5 character abbreviation of the component: `bd` board, `frm` form, `side` sidebar, `step` process/roadmap, `pv` privacy, and so on. Add new prefixes freely, but keep them short and reuse them once coined.
- Nesting deeper than two words is allowed when genuinely needed (`bd_item_tt`), but prefer flattening — if you need three levels the component is probably two components.

### Child-element abbreviation dictionary

The whole point is that the same meaning always gets the same abbreviation, everywhere, in both projects. Reading `_tt` should never require checking which component you are in.

| Structure | Text | Elements |
|---|---|---|
| `hd` header · `bd` body · `ft` footer | `tt` title · `sub` subtitle | `ic` icon · `img` image |
| `box` box · `wrap` wrapper | `lb` label · `val` value | `thumb` thumbnail · `btn` button |
| `item` item · `list` list | `desc` description · `txt`/`tx` body text | `link` link · `nm` name |
| `row` row · `grp` group | `meta` meta · `info` notice | `num` number · `dt` date |
| `sec` section · `b` block | | `cat` category · `vw` view count · `tag` tag |

Title is `tt` anywhere. Body is `bd` anywhere. If you need a meaning the table does not cover, coin a short abbreviation and then use it consistently for the rest of the work — and say which one you coined so it can be added to the dictionary.

Examples: `bd_list`, `bd_item`, `bd_tt`, `bd_dt`, `bd_vw`, `frm_lb`, `frm_val`, `side_tt`, `step_num`, `card_thumb`, `ft_policy`.

### Keep child names identical across variants

When a component has variants (`bd_st1`, `bd_st2`, …), a child that means the same thing keeps the **same class name in every variant**. The table row's title and the card's title are both `bd_tt`. Renaming per variant is what makes someone grep for a class and find three unrelated things.

## 3. Modifiers, states, and utilities

Naming the component is only half of it. Most drift happens in the three class families that sit on top of it — one page writes `.tab.on` and another writes `.tab.active`, and now the JS has to know which page it is on.

### Modifiers — variants of one component

| Kind | Form | Use when |
|---|---|---|
| Index | `bd_st1` · `bd_st2` · `step_flow4` | Variants have no natural name — they are just "layout 1, layout 2". Number them in order and keep numbering up. |
| Semantic | `btn_pri` · `btn_navy` · `btn_line` · `btn_lg` | The variant has a real meaning (color, weight, size). |
| Combined | `.bd_list.bd_st1` · `.agree.acc` | Base + variant are **both** present as classes, so shared styling lives on the base. |

Write the base variant explicitly too. `.bd_list.bd_st1` rather than a bare `.bd_list` for the default — otherwise variant 1 is the only one styled by a different selector shape than its siblings, and the shared rules end up split.

### States — shared, prefix-less, single words

States are the one place where a class carries no component prefix, because the same word must work on every component and in shared JS:

```
active   선택·현재 위치      open   펼침
in       reveal 등장 완료    on     켜짐
done     완료               show   노출
err      오류
```

Used in combination: `.bd_item.active`, `.side_dep.open`, `.step_item.done`.

Never coin a synonym. `on`, `active`, and `open` already mean three different things here — adding `selected` or `current` for a fourth shade of the same idea is exactly the drift this list prevents.

### Utilities — small, composable, orthogonal

```
여백    m_b20 · p_t30            (m/p + t/b/l/r + 숫자)
정렬    txt_center · f_between   (텍스트 정렬 / flex justify)
색      color_info · color_info2
```

Utilities are for one-off adjustments that do not deserve a component rule. If the same utility combination repeats across three or more places, that is a component, not a utility run — name it.

## 4. HTML authoring

### Keep tag depth minimal

Every wrapper `div` is a line someone has to read past. Modern CSS removes most of them:

- Use `grid-template-areas` on the component instead of wrapping text and image groups in layout `div`s.
- Menus are `nav > a` directly. `ul > li > a` for a horizontal menu bar is three tags doing one tag's job.
- Card links are `a.card` directly, not `article > a`.
- `clip-path: inset(0 round Xpx)` rounds an image without a thumbnail wrapper `div`.

### Use pseudo-elements for anything that is not content

| Situation | Avoid | Prefer |
|---|---|---|
| Ordinal numbers | `<span>1</span>` | `li::before` with `counter()` |
| Arrows, decorative characters | typed into the text | `a::after { content: " ›"; }` |
| Divider lines, ornaments | a separate `<div>` | `::before` / `::after` |

Decoration in the markup becomes decoration a screen reader announces and a translator translates. Keep it in CSS.

### Links

- When an image and its text are one piece of content, wrap them in a single `<a>`.
- Avoid a separate `<a>` per element inside the same card — it makes the same destination three tab stops.

### Lists and tables

- Content that is a list is `ul`/`ol` + `li`. Use `ol` when the order carries meaning.
- Table-shaped UI must fall back to a list layout below a breakpoint. A table that scrolls sideways on a phone is a broken table.

## 5. CSS authoring

### Declaration format — one line per rule

Group related properties by meaning and keep the block on one line. The expanded one-property-per-line style is not used: it turns a 6-property component into 8 lines and pushes the next component off the screen.

```css
/* preferred */
.stat { padding: 20px 24px; border-radius: 10px; display: flex; align-items: center; gap: 18px; }

/* avoid */
.stat {
  padding: 20px 24px;
  border-radius: 10px;
  display: flex;
}
```

Long rules may wrap onto a second line, still grouped by meaning (box model / layout / typography / color).

### `background` shorthand

Combine `background-image` + `background-size` + `background-position` + `background-repeat`:

```css
/* preferred */
.side_tt { background: url('/img/bg1.jpg') no-repeat center / cover; }

/* avoid */
.side_tt { background-image: url('/img/bg1.jpg'); background-size: cover; background-position: center; }
```

**Split them apart in exactly these cases**, because the shorthand resets `background-color`:

- Swapping only `background-image` on `:hover`.
- A base rule carrying `background-color` while a child or hover state carries `background-image` (control buttons, quick-menu icons, SNS icons, and similar).

Inside `:hover`, always write `background-image:` — never `background:`.

### Shared defaults via `parent > *`

When the same property repeats across three or more children, declare the default once on `parent > *` and override only what differs. Otherwise a padding change means editing five rules and missing one.

```css
/* preferred */
.bd_item > * { padding: 20px 10px; font-size: 15px; text-align: center; }
.col_tt { padding: 20px 14px; font-size: 17px; text-align: left; }

/* avoid */
.col_num  { padding: 20px 10px; font-size: 15px; text-align: center; }
.col_cat  { padding: 20px 10px; font-size: 15px; text-align: center; }
.col_auth { padding: 20px 10px; font-size: 15px; text-align: center; }
```

### Selector scoping

A class that only exists inside one component is written with its parent in front — inside media queries too. Without it, a search for `.col_num` gives no clue which component owns it, and a second component reusing the name silently collides.

```css
/* preferred */
@media (max-width: 768px) {
  .bd_item .col_num { display: none; }
  .bd_item .col_cat { order: 1; }
}

/* avoid */
@media (max-width: 768px) {
  .col_num { display: none; }
}
```

### Descendant selectors vs. classes

Use a descendant selector when the tag and structure are fixed and the scope is small; keep a class when siblings differ in style or the piece may be reused elsewhere.

```css
/* descendant — fixed small structure */
.stat_val strong { }
.stat_nm small { }
.gnb_mn > a::after { }
.ft_policy a:hover { }

/* class — siblings styled differently, or reusable */
.card_bdg { }
.card_tm { }
.ftag.hwp { }
```

### `:where()` for every default — specificity 0

Base styles in `basic.css` are wrapped in `:where()` so their specificity is zero and any single class can override them.

```css
/* basic.css — specificity 0, so `.srch_inp { border: none; }` is enough to override */
:where(input[type='text'], select, textarea) { height: 48px; border: 1px solid var(--btn-border); padding: 0 16px; }
```

- Bare `input[type='text']` is (0,1,1) — higher than a class (0,1,0) — so it overrides component styles by accident. Always wrap it.
- The global reset must be `:where()` too: `:where(body, body *, input, …) { margin: 0; padding: 0; box-sizing: border-box; }`. A bare `body *` is (0,0,1) and beat the `:where()` form defaults, which is how input text once ended up glued to the left edge. Every default = specificity 0; control comes from source order and classes only.

Defaults to assume: inputs, selects and textareas have **16px left/right inner padding**; radios and checkboxes are already custom-rendered at **22px** with their own border and checked colors. Do not redeclare their size or color in a component.

### Never redeclare what the reset already handles

| Property | Already handled by |
|---|---|
| `list-style: none` | `ul, ol, li { list-style: none }` |
| bare `margin: 0` / `padding: 0` | `body *` reset (positional values like `margin: 0 auto` are fine) |
| `cursor: pointer` on `<button>` | `button { cursor: pointer }` |
| `border: none` / `border: 0` on `<button>` | `button { border: 0 }` |
| `background: none` on `<button>` | `button { background: none }` |
| `align-items: center` on `<button>` | `button { display: inline-flex; align-items: center }` |
| `justify-content: center` on `<button>` | `button { justify-content: center }` |
| `outline: none` on text inputs, textarea, select | reset |
| `font-family: inherit` on input/button/select | reset sets font-family directly |
| `font-size` / `font-weight` on button/input/select equal to the reset value | only declare when overriding with a different value |

`<a>` does not get the button reset — declare `display: flex`, `align-items`, `justify-content` yourself when an anchor needs them.

If you are unsure what this project's reset actually covers, read its `basic.css` (or equivalent) before assuming — the table above describes the house baseline, and a project that has not adopted it needs these declarations written out. The `css-architecture` skill carries the reference baseline this table is derived from.

### Reserve space for absolutely positioned children with parent padding

When a container holds absolutely positioned children (icon buttons, badges), reserve their area explicitly with the parent's `padding`, then place the children inside that reserved band. The text then never runs under the button at any width.

```css
.srch_box { position: relative; padding: 0 85px 0 16px; }  /* right 85px = clear + submit buttons */
.srch_clr { position: absolute; left: calc(100% - 82px); }
.srch_btn { position: absolute; right: 10px; }
.srch_inp { width: 100%; }
```

## 6. Images

- Icons are CSS `background-image`, not `<img>`, whenever a class can identify them. `<img>` is for content images.
- Hover image swaps use the off/on `background-image` pattern, kept separate from `background-color` (see the shorthand exception above).
- Never use the `background` shorthand in a `:hover` rule — it wipes the base `background-color`.
- Image paths are project **absolute paths**, never relative (`/<project-root>/img/...`, e.g. `/portal/img/ic_search.svg`). Use the path prefix the project already uses; ask if it is not evident from the surrounding code.

## Before you hand code over

Run through this quickly — these are the mistakes that actually recur:

1. Any `-` in a class name? Any uppercase?
2. Same meaning spelled two ways (`title` here, `tt` there)?
3. A new state word that duplicates `active` / `open` / `on` / `done`?
4. A wrapper `div` that only exists to group two elements — can grid areas or a pseudo-element remove it?
5. Any expanded multi-line declaration block left behind?
6. Any `list-style: none`, `cursor: pointer` on a button, or bare `margin: 0` that the reset already does?
7. Any component class inside a media query without its parent in front?
8. `background:` shorthand inside a `:hover`?
9. Does the table-shaped UI have a mobile list fallback?

`references/patterns.md` has full worked markup for the recurring components (board list with mobile fallback, search box, card grid, two-column sub page, GNB, footer). Read it when building one of those rather than re-deriving the structure.
