---
name: web-publishing-workflow
description: How to plan and sequence a multi-page web publishing job BEFORE writing code — build the shared foundation first, then page structure, then responsive, then animation; present a reuse mapping and get it confirmed before coding; extract what a static mockup cannot show (motion, interaction, states). Use when a job is being scoped rather than typed: a new site or page set is starting, a design mockup or PDF has just been handed over, the user asks what order to work in, asks for a plan, or asks how to hand a design over. Equally for the other side of that handover — writing or reviewing a spec, a screen definition, or a requirements list that a publisher or developer will build from, where the four things a static mockup cannot show (motion, interaction, states, live behavior) are what a spec routinely omits. Not for the act of writing markup — that is the conventions skill. [한글] 코드를 쓰기 전 계획·순서·소통 방식. 만드는 쪽과 넘기는 쪽 모두에 해당. "새 사이트", "페이지 여러 개", "시안 받았어", "시안 줄게", "이거 어떻게 작업하지", "작업 순서", "계획 먼저", "구조부터 보여줘", "어디부터 해야 해", "견적", "일정" 요청에 사용. 기획 쪽에서는 "기획서 쓰는데", "요건 정리", "화면 정의서", "스토리보드", "개발자한테 넘길 문서", "퍼블리셔한테 전달", "이 정도면 전달해도 될까", "빠진 거 없나" 요청에 사용 — 시안에 담기지 않는 움직임·인터랙션·상태·실제 동작이 기획서에서 흔히 빠지는 항목이므로 그 확인에 쓴다. 마크업을 실제로 작성하는 단계가 아니라, 그 전에 무엇을 어떤 순서로 만들지 정하는 단계에 적용.
---

# Web publishing workflow

## 요약 (Korean summary)

코드를 쓰기 **전** 단계의 규칙입니다. 실제 마크업·CSS 작성 규칙은 `html-layout-conventions` 스킬에 있습니다.

| 절 | 핵심 내용 |
|---|---|
| 1. 가장 큰 리스크 | 다중 페이지의 실패는 코드 품질이 아니라 **일관성 붕괴**. 페이지마다 헤더 높이·분기점·모션 속도가 따로 노는 것을 막는 게 목적. |
| 2. 레이어 순서 | 페이지를 하나씩 끝내지 말고 **공통기반 → 페이지구조 → 반응형 → 애니메이션** 순으로 가로로 쌓는다. |
| 3. 코딩 전 매핑 | 새 페이지·큰 변경은 "재사용 매핑 + 작업 계획"을 먼저 제시하고 **확인받은 뒤** 코딩한다. |
| 4. 시안에 없는 정보 | 시안은 정지된 한 순간. 움직임·인터랙션·상태 변화·실제 동작은 이미지에 없다. 받는 쪽은 **코딩 전에 물어보고**, 넘기는 쪽은 **기획서에 적는다**. |
| 5. 임의 진행 금지 | 확신 없는 결정, 범위 밖 작업은 멋대로 하지 않는다. 디자인은 유지하고 구조만 규약에 맞춘다. |
| 6. 도구 분리 | 한 프로젝트는 한 도구로. 도구끼리 작업 내용을 주고받지 못하므로 섞으면 앞 작업이 이어지지 않는다. |

---

## 1. What actually goes wrong

The risk in a multi-page site is not bad code. It is **drift**: page 3 has a 90px header and page 7 has 96px, one page breaks at 768px and another at 800px, cards fade in over 0.4s here and 0.7s there. Every rule below exists to stop that, and the cost of drift is paid at the end — when fixing it means touching every page.

So the sequencing rule is: **finish nothing vertically until the horizontal layer below it is fixed.**

## 2. Layer order — build across, not down

Do not take one page all the way to done and then start the next. Stack layers across all pages.

### Layer 1 — Foundation (fix this before any page)

Everything every page will share, decided once and frozen:

- **Design tokens** — colors, font sizes, spacing scale, radius, shadow. One place.
- **Breakpoints** — the actual numbers. Pick them once; no page invents an intermediate value.
- **Shared components** — header, footer, GNB, buttons, cards.
- **Motion defaults** — base duration and easing curve.

This layer is the constitution. When a later page needs a value that is not in it, that is a signal to amend the foundation, not to hardcode locally.

### Layer 2 — Page structure and layout

Build each page's structure using layer 1. State explicitly, per page, which tokens and which shared components are being reused — this is what the mapping in §3 is for.

### Layer 3 — Responsive

**Fix the desktop layout first, then do that page's breakpoints as a separate pass.** Trying to satisfy desktop and mobile in one pass produces rules that fight each other. Because layer 1 fixed the breakpoints, every page folds at the same widths.

### Layer 4 — Animation, last

Structure and responsive must be settled first. Adding motion to a layout that is still moving means every structural edit breaks the animation and you do the work twice.

Order: **skeleton → clothes → movement.**

### Multi-page sets

For a set of similar pages (5-step signup, a board family, policy pages): build **one template page, get it confirmed, then clone it** and swap only the fields and text. Never build five in parallel from the same spec — five interpretations is exactly the drift this skill exists to prevent.

## 3. Reuse mapping before the build

Before writing a new page or making a large change, present this and **wait for confirmation**:

```
재사용 매핑
  페이지 제목      → cont_hd / cont_tt          (기존 그대로)
  좌측 메뉴        → sidebox / side_nav          (기존 그대로)
  목록             → bd_list.bd_st1              (기존 그대로)
  상단 통계 3개    → 신규 필요 — stat 계열로 제안, 확인 부탁

작업 계획
  1. 템플릿 1개 (목록) 완성 → 확인
  2. 나머지 3개 복제
  3. 반응형 768/480
```

The point is to surface the mismatch **before** the code exists, when it costs one message instead of a rewrite. Two rules make it work:

- **Reuse beats invention.** If a component already exists for this shape, use it. Do not coin a new class because the new page's version is 4px different — that difference is a modifier, not a new component.
- **Ignore whatever placeholder UI is already sitting in the page.** Work from the design, not from whatever was left there.

Components can also be lifted from a sibling project in the same repo — copy and adapt. Point at the source explicitly ("the X from project Y") rather than re-deriving it.

## 4. What a mockup cannot tell you

A mockup is one frozen instant. These four things are never in it, and guessing at them is where rework comes from.

The same four work in both directions. **Building from a spec:** ask these before coding. **Writing a spec:** these are what to put in it, because they are what specs routinely omit and what the person building will come back to ask.

| Missing | Ask / state it |
|---|---|
| **Motion** | Does anything animate on scroll-in? On page enter? On hover? |
| **Interaction** | Where does this button go? Does this accordion open? What happens on submit? |
| **States** | Hover / disabled / error / active — the mockup shows only the default state. |
| **Live behavior** | Carousel autoplay, infinite scroll, polling, count-up. |

When reviewing a spec or a mockup handover before it goes out, run the four as a checklist and name what is missing rather than assuming the builder will ask.

Also confirm the constraints, because they change the approach rather than the details:

- **Fidelity** — pixel-exact, or "close enough, fast"?
- **Stack** — plain HTML/CSS, or a framework? Which browsers? Web fonts or local files?
- **Responsive mockups** — is there a mobile mockup, or only desktop? If only desktop, ask in words how it should fold. Without that, the fold is a guess.

When receiving mockups: **Original-resolution PNG, split by section.** A blurry full-page screenshot loses the font sizes, the spacing, and the colors — which are most of the job.

## 5. Working rules

- **Do not proceed on a guess.** A decision you are not sure about, or work outside the stated scope, gets raised rather than silently made.
- **Design stays, structure changes.** When refactoring someone else's markup into the house conventions, keep the visual design identical. If the design has to change, explain why first.
- **Ask when the value is not in the code.** Path prefixes, token names, breakpoint numbers — if the project already uses one, follow it; if it is not evident, ask rather than inventing.

## 6. One project, one tool

Coding agents do not share context with each other. Running two on the same project loses the handoff and produces file conflicts.

- Keep the structural work — the part that reads the whole codebase and holds the conventions — in **one** agent from start to finish.
- A second tool is reasonable only for the last step: opening the result in a browser and nudging it against the mockup by eye (animation timing, mobile spacing). Code alone cannot settle those.
- If you do split, **split by file range, not by task** — "this section is the other tool's" — so the two never write the same file.

## Before you start coding, check

1. Are the breakpoint numbers fixed, or still "mobile-ish"?
2. Do design tokens exist, or is the first page about to hardcode colors?
3. Has the reuse mapping been shown and confirmed?
4. Are motion, interaction, and states known — or being assumed?
5. Is this a page set that should start from one confirmed template?
6. Is the desktop layout settled before responsive work begins?
