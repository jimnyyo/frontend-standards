# Reference reset — `basic.css`

The baseline the conventions assume. Every declaration here is something a component must **not** re-declare.

Two hard rules govern the whole file:

1. **Anything that is a default is wrapped in `:where()`** so its specificity is (0,0,0) and any single class overrides it.
2. **Anything that is a deliberate house decision** — not a default, but a choice the site actively makes, like the custom checkbox rendering — may be a bare selector, because it is meant to hold unless a component overrides it on purpose.

Values below (48px input height, 22px controls, 16px inner padding) are the house baseline. Change the numbers for a project if needed, but keep the structure and the `:where()` wrapping.

---

## 1. Reset

```css
html, body { overflow-x: clip; }

/* 특이도 0 — 이 줄이 :where() 없이 `body *` 로 쓰이면 폼 기본 padding 을 덮어써 인풋 글자가 좌측에 붙는다 */
:where(body, body *, input, textarea, button, select) { margin: 0; padding: 0; box-sizing: border-box; }

body, input, textarea, select, button { font-family: var(--font_main, system-ui, sans-serif); font-size: 17px; font-weight: 400; line-height: 1.3; letter-spacing: -0.02em; }

table { padding: 0; border-collapse: collapse; border-spacing: 0; }
img { max-width: 100%; height: auto; vertical-align: top; }
img, fieldset, input[type='radio'], input[type='checkbox'] { border: 0; }
iframe { display: block; margin: 0 auto; padding: 0; border: 0; resize: none; }

a { color: inherit; text-decoration: none; letter-spacing: normal; outline: none; -webkit-tap-highlight-color: transparent; }
a:link, a:visited, a:active, a:focus, a:hover { text-decoration: none; }

button { display: inline-flex; align-items: center; justify-content: center; border: 0; background: none; cursor: pointer; }
ul, ol, li { list-style: none; }
em, address { font-style: normal; }

:where(input[type='text'], input[type='email'], input[type='password'], input[type='tel'], input[type='number'], input[type='date'], input[type='url'], input[type='search'], textarea, select) { outline: none; }

input:disabled, input[readonly] { background-color: #f6f7f8; }

* { -webkit-text-size-adjust: none; }   /* 화면 회전·폰트 확대 시 자동 리사이즈 방지 */
```

### Screen-reader / hidden helpers

```css
.hidd { position: absolute !important; top: -20000px !important; left: -50000px !important; width: 1px !important; height: 1px !important; overflow: hidden !important; text-indent: -3000em; visibility: hidden; }
```

---

## 2. Design tokens

```css
:root {
  /* Font sizes */
  --font14: 14px; --font15: 15px; --font16: 16px; --font17: 17px;
  --font18: 18px; --font20: 20px; --font22: 22px; --font24: 24px;
  --font26: 26px; --font30: 30px; --font32: 32px; --font36: 36px;

  /* Brand */
  --color-primary:   #16408d;
  --color-secondary: #246beb;
  --color-accent:    #0089cf;
  --color-gray:      #717171;

  /* Text — 역할 기준 (색상값 기준 아님) */
  --color-text-title:       #1d1d1d;
  --color-text-body:        #555555;
  --color-text-small:       #717171;
  --color-text-placeholder: #8e8e8e;
  --color-text-disabled:    #8e8e8e;

  /* State */
  --color-info:    #f26522;
  --color-danger:  #f26522;
  --color-warning: #fcaf17;
  --color-success: #246beb;

  /* Button */
  --btn-bg:           #16408d;
  --btn-bg-hover:     #246beb;
  --btn-bg-disabled:  #e4e4e4;
  --btn-border-hover: #717171;

  /* Background — 층위 기준 */
  --bg_color:  #fff;
  --bg-layer1: #f8f8f8;
  --bg-layer2: #eff5fd;
  --bg-layer3: #f3f8fe;
  --bg-layer4: #fff9f6;
  --bg-hover:  rgba(36, 107, 235, 0.06);

  /* Semantic */
  --font_main:    'Pretendard', 'Apple SD Gothic Neo', sans-serif;
  --text_color:   #1d1d1d;
  --border_color: #c6c6c6;
  --transition:   all 0.2s ease;
  --shadow:       0 2px 8px rgba(0, 0, 0, 0.08);
  --shadow2:      0 5px 16px rgba(0, 65, 132, 0.15);
  --radius:       5px;
  --card_radius:  6px;
  --thumb_radius: 4px;
}
```

---

## 3. Form defaults — all `:where()`

Wrapped so any component class overrides them with a single declaration.

```css
:where(input[type='text'], input[type='email'], input[type='password'], input[type='tel'], input[type='number'], input[type='date'], input[type='url'], input[type='search'], select, textarea) {
  width: 100%; height: 48px; padding: 0 16px;
  border: 1px solid var(--border_color); border-radius: 8px;
  color: var(--color-text-title); background: #fff;
  transition: border-color 0.15s;
}

:where(textarea) { height: auto; min-height: 120px; padding: 14px 16px; line-height: 1.6; resize: vertical; }

:where(select) { padding-right: 36px; background: #fff url('/<project>/img/arrow_down.png') no-repeat right 12px center / 7px auto; appearance: none; cursor: pointer; }

:where(input[type='text'], input[type='email'], input[type='password'], input[type='tel'], input[type='number'], input[type='date'], input[type='url'], input[type='search'], select, textarea):focus {
  border-width: 2px; border-color: var(--color-primary);
}

:where(input, textarea)::placeholder { color: var(--color-text-placeholder); }
```

> `--border_color` 나 `--color-primary` 처럼 토큰을 참조하므로, 토큰 블록이 이 규칙들보다 **앞에** 있어야 한다.

---

## 4. Radio & checkbox — custom rendered at 22px

These are bare selectors, not `:where()`, because the custom rendering is a deliberate decision rather than a fallback default. **Components must not redeclare their size or colors.**

```css
:where(input[type='radio'], input[type='checkbox']) { width: 22px; height: 22px; flex-shrink: 0; cursor: pointer; }

input[type='checkbox'] {
  appearance: none; -webkit-appearance: none;
  width: 22px; height: 22px; flex-shrink: 0;
  border: 1px solid var(--border_color); border-radius: 4px;
  background: #fff url('/<project>/img/check.png') no-repeat center;
  cursor: pointer; transition: border-color 0.15s;
}
input[type='checkbox']:checked,
input[type='checkbox']:checked:hover { border-color: var(--btn-bg-hover); background-image: url('/<project>/img/check_on.png'); }

/* 변형 — 체크 시 파란 채움 + 흰 체크 */
input[type='checkbox'].chk_fill { width: 24px; height: 24px; }
input[type='checkbox'].chk_fill:checked,
input[type='checkbox'].chk_fill:checked:hover { border-color: var(--color-secondary); background: var(--color-secondary) url('/<project>/img/check_on_white.png') no-repeat center; }

input[type='radio'] {
  appearance: none; -webkit-appearance: none;
  width: 22px; height: 22px; flex-shrink: 0;
  border: 1px solid var(--border_color); border-radius: 50%;
  background: #fff; cursor: pointer; transition: border-color 0.15s;
}
input[type='radio']:checked { border-color: var(--color-secondary); background: radial-gradient(circle, var(--color-secondary) 45%, #fff 47%); }
```

> `:checked:hover` 를 `:checked` 와 같이 선언해 두는 이유: hover 규칙이 따로 있으면 체크된 상태에 마우스를 올렸을 때 체크 이미지가 사라진다.

---

## 5. Validation error state

Add `class="err"` to a field that failed validation. Nothing else is needed.

```css
input.err, select.err, textarea.err { border: 2px solid #ea0202; color: #ea0202; }
input.err::placeholder { color: #ea0202; }
input.err:focus, select.err:focus, textarea.err:focus { border-color: #ea0202; }
```

---

## 6. Responsive form sizing

```css
@media (max-width: 768px) {
  :where(input[type='text'], input[type='email'], input[type='password'], input[type='tel'], input[type='number'], input[type='search'], select) { height: 44px; }
  :where(textarea) { min-height: 100px; }
}

@media (max-width: 480px) {
  input, select, textarea { font-size: 15px; }   /* 요소 선택자 — 클래스로 지정한 값은 그대로 우선 */
}
```

---

## What this file means for component CSS

Because the above exists, **do not write these again** in any component stylesheet:

| Declaration | Already handled by |
|---|---|
| `list-style: none` | `ul, ol, li` |
| bare `margin: 0` / `padding: 0` | the `:where()` reset (positional values like `margin: 0 auto` are fine) |
| `cursor: pointer` on `<button>` | `button` |
| `border: none` / `border: 0` on `<button>` | `button` |
| `background: none` on `<button>` | `button` |
| `display: inline-flex` / `align-items` / `justify-content: center` on `<button>` | `button` |
| `outline: none` on text inputs, textarea, select | reset |
| `font-family: inherit` on input/button/select | reset sets `font-family` directly |
| `font-size` / `font-weight` on button/input/select equal to the reset value | only declare when the value differs |
| radio / checkbox size, border, or checked color | §4 above |
| input inner padding of `0 16px` | §3 above |

`<a>` does **not** get the button reset — declare `display: flex`, `align-items`, `justify-content` yourself when an anchor needs them.
