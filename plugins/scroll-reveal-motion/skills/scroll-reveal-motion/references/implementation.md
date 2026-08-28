# Reference implementation — scroll reveal & step flow

Complete working JS and CSS. Copy into the shared script and shared stylesheet.

---

## 1. `initReveal()`

```js
/* ============================================================
   스크롤 리빌 — 스크롤 진입 시 페이드인 + 슬라이드
   ① 그룹 모드: 부모 [data-reveal] → 직계 자식 자동 .reveal 처리
   ② 개별 모드: class="reveal" (+ 방향 클래스, data-reveal-delay)
   ⚠️ threshold 는 0 고정. 타이밍 조절은 rootMargin 하단 음수값으로만.
   ============================================================ */
function initReveal() {
  const DIRS = ['left', 'right', 'down', 'zoom'];

  /* 그룹 모드 — 부모 속성을 직계 자식에 펼침 */
  document.querySelectorAll('[data-reveal]').forEach(group => {
    const dir  = group.getAttribute('data-reveal').trim();
    const step = parseInt(group.dataset.revealStagger, 10) || 0;
    Array.from(group.children).forEach((child, i) => {
      child.classList.add('reveal');
      if (DIRS.includes(dir)) child.classList.add(dir);
      if (step && !child.dataset.revealDelay) child.dataset.revealDelay = i * step;
    });
  });

  const targets = document.querySelectorAll('.reveal');
  if (!targets.length) return;

  /* 동작 줄이기 설정이거나 IntersectionObserver 미지원 → 관찰 없이 즉시 표시.
     리빌은 기본 상태가 opacity:0 이므로, 이 분기가 없으면 콘텐츠가 영영 안 보인다. */
  const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (reduce || !('IntersectionObserver' in window)) {
    targets.forEach(el => el.classList.add('in'));
    return;
  }

  const io = new IntersectionObserver((entries, obs) => {
    entries.forEach(e => {
      if (!e.isIntersecting) return;
      const delay = e.target.dataset.revealDelay;
      if (delay) e.target.style.animationDelay = delay + 'ms';
      e.target.classList.add('in');
      obs.unobserve(e.target);          /* 1회 실행 후 관찰 해제 */
    });
  }, {
    threshold: 0,                        /* ⚠️ 고정. 올리면 화면보다 긴 요소가 비율 미달로 영영 안 뜬다 */
    rootMargin: '0px 0px -18% 0px'       /* 등장 타이밍은 여기서만 조절 (음수 클수록 늦게) */
  });

  targets.forEach(el => io.observe(el));
}
initReveal();
```

### Timing reference

| `rootMargin` bottom | Effect |
|---|---|
| `0%` | Fires the instant the top edge enters the viewport |
| `-10%` | Slightly delayed |
| `-18%` | Default — comfortably inside the viewport |
| `-30%` | Element must reach roughly mid-screen |

Never substitute `threshold` for any of these.

---

## 2. Reveal CSS

```css
/* 기본 상태 — 보이지 않음. 그래서 관찰이 실패하면 콘텐츠가 사라진다는 점에 주의 */
.reveal { opacity: 0; }

.reveal.in       { animation: rv_up 0.7s ease-out both; }
.reveal.left.in  { animation-name: rv_left; }
.reveal.right.in { animation-name: rv_right; }
.reveal.down.in  { animation-name: rv_down; }
.reveal.zoom.in  { animation-name: rv_zoom; }

@keyframes rv_up    { from { opacity: 0; transform: translateY(40px); }  to { opacity: 1; transform: none; } }
@keyframes rv_left  { from { opacity: 0; transform: translateX(-40px); } to { opacity: 1; transform: none; } }
@keyframes rv_right { from { opacity: 0; transform: translateX(40px); }  to { opacity: 1; transform: none; } }
@keyframes rv_down  { from { opacity: 0; transform: translateY(-40px); } to { opacity: 1; transform: none; } }
@keyframes rv_zoom  { from { opacity: 0; transform: scale(0.92); }       to { opacity: 1; transform: none; } }

/* 접근성 — JS 분기와 이중으로 건다 */
@media (prefers-reduced-motion: reduce) {
  .reveal { opacity: 1 !important; animation: none !important; }
}
```

`animation: … both` matters: `both` holds the `from` state before the delay elapses, so a staggered child does not flash at full opacity before its turn.

### Responsive exception

```css
/* 데스크탑에서 히어로 위에 떠 있는 박스 — 항상 노출이므로 리빌 무효화.
   translateY 가 absolute 위치와 충돌해 자리가 어긋나는 것을 막는다. */
@media (min-width: 901px) {
  .rgn.reveal { opacity: 1; animation: none; }
}
```

---

## 3. `initStepFlow()`

```js
/* ============================================================
   추진체계 스텝 플로우 (.step_flow) — 가변 줄바꿈 대응
   같은 줄에 다음 카드가 있을 때만 .has_arrow → 줄 끝엔 화살표 없음
   ============================================================ */
function initStepFlow() {
  const flows = document.querySelectorAll('.step_flow');
  if (!flows.length) return;

  const update = () => flows.forEach(flow => {
    const steps = Array.from(flow.children);
    steps.forEach((s, i) => {
      const next = steps[i + 1];
      s.classList.toggle('has_arrow', !!next && next.offsetTop === s.offsetTop);
    });
  });

  update();
  window.addEventListener('resize', update);
}
initStepFlow();
```

`offsetTop` equality is the only reliable "same visual row" test here — flex/grid wrapping is not queryable in CSS, and `getBoundingClientRect().top` brings sub-pixel noise that breaks equality comparison.

### Step flow markup & CSS

```html
<ol class="step_flow">
  <li><span class="step_num">01</span><p class="step_tt">신청 접수</p></li>
  <li><span class="step_num">02</span><p class="step_tt">서류 심사</p></li>
  <li><span class="step_num">03</span><p class="step_tt">선정 통보</p></li>
</ol>
```

```css
.step_flow { display: flex; flex-wrap: wrap; gap: 56px 28px; }
.step_flow > * { position: relative; flex: 1 1 220px; padding: 28px 20px; border-radius: var(--radius); background: var(--bg-layer2); text-align: center; }

/* 화살표 — 같은 줄에 다음 카드가 있는 요소에만 JS가 .has_arrow 를 붙인다 */
.step_flow > .has_arrow::after {
  content: ''; position: absolute; right: -28px; top: 50%; transform: translateY(-50%);
  width: 28px; height: 100%;
  background: url('/<project>/img/arrow_step.svg') no-repeat center / 10px auto;
}

@media (max-width: 768px) { .step_flow > * { flex: 1 1 calc(50% - 14px); } }
@media (max-width: 480px) { .step_flow { gap: 28px; } .step_flow > * { flex: 1 1 100%; } }
```

The arrow is a pseudo-element, never markup — a `<span>` arrow would be read aloud by a screen reader and picked up by translation.

To render a different procedure, replace the contents of each `<li>`. The wrapping logic and arrow behavior are independent of what is inside.

---

## 4. Common failures

| Symptom | Cause |
|---|---|
| A long section never appears | `threshold` was raised above 0. The element is taller than the viewport and can never reach the ratio. |
| Nothing appears anywhere | The observer never ran (script error before `initReveal`, or an unsupported browser without the fallback branch). Base state is `opacity: 0`. |
| Animation replays on scroll up | `unobserve` missing. |
| Staggered items flash before their delay | `animation-fill-mode` is not `both`. |
| A floating element jumps into the wrong place | Reveal's `transform` is fighting `position: absolute`. Disable reveal at that breakpoint. |
| Arrow points off the end of a row | `initStepFlow` never re-ran after a resize, or arrows are hardcoded in the markup. |
