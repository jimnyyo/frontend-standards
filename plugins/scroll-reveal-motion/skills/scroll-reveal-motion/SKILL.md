---
name: scroll-reveal-motion
description: Scroll-triggered entrance animation and step-flow connectors for static sites, done with IntersectionObserver and CSS classes — a `data-reveal` group attribute that targets direct children automatically with optional stagger, `threshold: 0` held fixed so elements taller than the viewport still fire, timing tuned only through negative bottom `rootMargin`, `prefers-reduced-motion` short-circuiting to visible, one-shot observation, and arrows that appear between step cards only when they share a row. Use when adding or debugging scroll entrance effects, when elements fail to appear on scroll, when motion timing needs adjusting, or when building a numbered process/roadmap row. [한글] 스크롤 진입 등장 모션과 추진체계(단계 플로우). "스크롤 애니메이션", "스크롤하면 나타나게", "페이드인", "등장 효과", "리빌", "요소가 안 나타나", "애니메이션 타이밍", "순차 등장", "추진체계", "프로세스 단계", "단계 화살표" 요청에 사용. 구조·반응형이 끝난 뒤 마지막 레이어에 적용.
---

# Scroll reveal & step flow

## 요약 (Korean summary)

구조와 반응형이 자리잡은 **뒤** 마지막에 입히는 레이어입니다. 작업 순서는 `web-publishing-workflow` 스킬을 참고하세요.

| 절 | 핵심 내용 |
|---|---|
| 1. 사용법 | 부모에 `data-reveal="left"` → 직계 자식 자동 적용. 개별은 `class="reveal"`. |
| 2. ⚠️ threshold 0 고정 | 올리면 **화면보다 긴 요소가 영영 안 뜬다.** 타이밍 조절은 `rootMargin` 하단 음수로만. |
| 3. 접근성 | `prefers-reduced-motion` 이면 애니메이션 없이 바로 표시. CSS·JS 양쪽에서. |
| 4. 1회 실행 | 등장 후 `unobserve` — 스크롤 위아래로 오갈 때 반복 재생되지 않게. |
| 5. 반응형 예외 | 데스크탑에서 항상 보이는 요소는 해당 구간에서 리빌을 무효화해야 위치가 안 깨진다. |
| 6. step_flow | 화살표는 **같은 줄일 때만** `.has_arrow`. 줄바꿈 지점엔 안 생긴다. 리사이즈마다 재계산. |

구현 전문(JS·CSS)은 `references/implementation.md`에 있습니다.

---

## 1. Usage

Two modes. Prefer the group form — it keeps the markup clean and the stagger consistent.

```html
<!-- ① 그룹 (권장): 부모에만 지정, 직계 자식이 자동 타깃 -->
<ul data-reveal="left" data-reveal-stagger="80">
  <li>…</li>   <!-- 0ms -->
  <li>…</li>   <!-- 80ms -->
  <li>…</li>   <!-- 160ms -->
</ul>

<!-- ② 개별: 방향 클래스를 직접 -->
<div class="reveal left">…</div>
<div class="reveal zoom" data-reveal-delay="200">…</div>
```

| Attribute | Meaning |
|---|---|
| `data-reveal="<dir>"` | Direction for all direct children. `left` · `right` · `down` · `zoom`. Omit the value for the default upward slide. |
| `data-reveal-stagger="<ms>"` | Per-child delay step. Child *n* gets `n × ms`. |
| `data-reveal-delay="<ms>"` | Explicit delay on one element. Set by the stagger loop, or written by hand. |

The class `in` is what the observer adds when an element enters — that is the state class the CSS animates on. It is the same shared `in` state used elsewhere in the conventions.

## 2. `threshold: 0` is not a tuning knob

This is the one rule in this skill that causes a bug you will not diagnose quickly.

```js
{ threshold: 0, rootMargin: '0px 0px -18% 0px' }
```

**`threshold` stays at 0. Permanently.**

`threshold: 0.3` reads as "fire when 30% of the element is visible", and it works fine while testing on a desktop with short cards. Then a long section — a terms-of-service article, a 20-item list, anything taller than the viewport — **can never reach 30% visibility**, because the viewport itself is smaller than 30% of the element. The observer never fires. The element stays at `opacity: 0` forever, and the page looks broken in a way that no error message explains.

So when the animation should fire later, **move `rootMargin`'s bottom value more negative** instead:

```js
rootMargin: '0px 0px -10% 0px'   /* 조금 일찍 */
rootMargin: '0px 0px -18% 0px'   /* 기본 */
rootMargin: '0px 0px -30% 0px'   /* 더 늦게 — 화면 중앙쯤 와야 등장 */
```

This shrinks the detection area from the bottom, so the element has to travel further up before crossing it. Timing changes; the guarantee that it eventually fires does not.

## 3. Reduced motion

Users who turn on "reduce motion" must get the content, not a stripped-down animation. Handle it in **both** layers, because they cover different failures:

```js
const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
if (reduce || !('IntersectionObserver' in window)) {
  targets.forEach(el => el.classList.add('in'));   /* 관찰 없이 즉시 표시 */
  return;
}
```

```css
@media (prefers-reduced-motion: reduce) {
  .reveal { opacity: 1 !important; animation: none !important; }
}
```

The JS branch also covers browsers with no `IntersectionObserver` — without it, those users see a permanently blank page rather than an un-animated one. This is the failure mode that makes reveal effects genuinely dangerous: **the default state is invisible**, so anything that stops the observer hides the content.

## 4. Fire once, then stop observing

```js
e.target.classList.add('in');
obs.unobserve(e.target);
```

Without `unobserve`, scrolling up and back down replays the animation every time, which reads as a glitch. It also keeps every element under observation for the life of the page for no reason.

## 5. Responsive exceptions

An element that is already visible at one breakpoint must not be reveal-controlled there. A floating box that sits in the hero on desktop but returns to normal flow on mobile is the common case: the reveal's `translateY` fights its absolute positioning.

Disable it for that range explicitly:

```css
@media (min-width: 901px) {
  .rgn.reveal { opacity: 1; animation: none; }   /* 데스크탑은 항상 노출 — 리빌 무효화 */
}
```

Rule of thumb: **reveal only what actually scrolls into view.** Anything visible at first paint should not depend on the observer at all.

## 6. Step flow — arrows only within a row

For a numbered process row (추진체계, roadmap, procedure), the arrow between cards must disappear at line wraps, or a dangling arrow points off the end of each row.

There is no CSS selector for "is on the same visual line", so it is measured:

```js
s.classList.toggle('has_arrow', !!next && next.offsetTop === s.offsetTop);
```

Same `offsetTop` means same row. The arrow itself is a pseudo-element on `.has_arrow`, so the markup stays clean:

```css
.step_flow > .has_arrow::after { content: ''; position: absolute; right: -28px; top: 50%; /* … */ }
```

Recalculate on `resize` — the wrap points move as the layout reflows, and a stale calculation leaves arrows in the wrong places after a rotate or a window drag.

The inner card is just markup: swap its contents and the same component renders any stepped procedure.

## Before you ship a motion pass, check

1. Is `threshold` still 0? Did someone raise it to fix timing?
2. Is there a section taller than the viewport that carries `reveal`? Scroll to it and confirm it appears.
3. Does the page render with motion disabled in OS settings?
4. Does anything replay when scrolling up and back down?
5. Is anything visible at first paint still waiting on the observer?
6. Do step arrows disappear at the wrap point at every breakpoint?
