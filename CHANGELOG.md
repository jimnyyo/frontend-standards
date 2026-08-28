# CHANGELOG

## marketplace 2.3.1 - 2026-08-28

### 변경 - `event-page-embed` 1.1.0 -> 1.1.1
"기존 페이지는 소급 적용하지 않는다"를 리빌 클래스 한 줄에서 **일반 규칙으로** 승격했습니다. 라이브 페이지를 열어 문구 하나 고칠 때 Claude가 구조나 클래스명을 규칙에 맞춰 바꾸자고 하는 것을 막기 위함입니다.

- 2절에 "These rules are for new pages" 항목 신설 - 라이브 페이지는 **요청받은 수정만**, 그 페이지가 쓰던 방식대로. 규칙과 충돌하면 페이지를 따르고 그 사실만 알린다
- 근거: CMS에 사는 페이지는 클래스명 하나 바꿔도 블록 전체를 다시 붙여넣고 두 템플릿에서 재검증해야 한다
- description과 요약표에도 명시

---

## marketplace 2.3.0 — 2026-08-28

### 변경 — `event-page-embed` 1.0.0 → 1.1.0
빠져 있던 두 가지를 채웠습니다.

**① 기존 규약이 어디까지 적용되는지 (2절 신설).** `html-layout-conventions`는 **그대로 적용**됩니다 — 제외한 건 파일 단위 스킬 둘뿐인데, 초판이 이를 명시하지 않아 전부 안 쓰는 것처럼 읽혔습니다.
- 그대로: 네이밍·약어 사전·상태 클래스·태그 깊이·한 줄 선언·`background` 축약·absolute 자식 padding
- 적용 안 함: 페이지 골격(`.wrap > .header + .gnb + .container`) — 사이트 골격이지 한 장짜리 페이지 골격이 아님
- **정반대로 뒤집힘: "리셋이 하는 선언은 다시 쓰지 마라"** — 삽입형 페이지엔 `basic.css`가 없으므로 `list-style:none`·`cursor:pointer`·`margin:0`을 래퍼 안에 직접 써야 한다. 스코프 리셋 예시 추가
- 새 페이지의 리빌 상태 클래스는 `in`으로 통일. 이미 `in-view`로 나간 페이지는 그대로 둔다

**② 호스트 header/footer 두 모드 (5절 신설).** 초판은 "전부 숨기기"만 다뤘는데 실제로는 두 방식을 쓰고 있었습니다. **레이아웃 짜기 전에 정해야** 하는 결정입니다.
- **A. 유지** — 호스트 컨테이너 폭 안에서 작업, full-bleed 금지(헤더와 폭이 어긋나 보임), 방해되는 호스트 부품만 콕 집어 숨기고 이유를 주석으로
- **B. 대체** — `:not()`으로 호스트 것만 숨기고 자체 제공, full-bleed 필요(`width:100vw; margin-left:calc(50% - 50vw)`), 호스트 `min-width`는 여전히 살아 있음
- 두 모드가 공유하는 규칙: 래퍼를 벗어나도 되는 선택자는 여기뿐

체크리스트 10 → 12항목, description에 두 모드와 규약 적용 범위 명시.

---

## marketplace 2.2.0 — 2026-08-28

### 신규 — `event-page-embed` 1.0.0
남의 CMS 본문 필드에 통째로 붙여 넣는 이벤트·캠페인 상세 페이지. `08/후원`(선교협력)과 `08/성결필사 영상추가` 두 작업에서 뽑아냈습니다.

**나머지 5종과 전제가 반대**라 별도 스킬로 분리했습니다. 파일을 서버에 올리는 게 아니므로 파일 분할·공통 CSS·include가 전부 불가능합니다. description에 `css-architecture`·`static-site-includes`를 적용하지 말라고 명시해 두었습니다.

- 삽입 블록 마커 — `.html`은 미리보기용 껍데기, 납품물은 마커 사이. `<style>`·`<script>`·`<link>` 전부 래퍼 안에
- 접두어 스코프 — 래퍼·클래스·CSS를 하나의 접두어로. 변수는 `:root`가 아니라 래퍼에
- 호스트 chrome 무력화 — `header:not(.<접두어>_header){display:none!important}`
- 특이도 대응 — 템플릿이 래퍼에 거는 `img` 규칙 등을 접두어를 겹쳐 이긴다. `!important` 금지
- 호스트 `min-width` 때문에 브라우저 창 줄이기가 유효한 모바일 테스트가 아닐 수 있음
- 본문 필드 공유 시 PC/MO — 스타일시트는 `media='not all'`로 끄고, `<picture><source media>`는 CSS로 못 끄므로 1×1 GIF로 무력화
- 폰트 서브셋 + 원본 CDN `src` 폴백 + 상단 폰트만 preload + 문구 수정 시 재생성 경고
- 호스트 존중 — 전역 콜백은 덮지 말고 체이닝, `document.currentScript.parentNode` 기준 조회, JS 없이도 내용 노출
- 날짜 리비전 파일명 (`index_260820.html`) — CMS에 사는 페이지는 파일 세트가 곧 이력
- `references/cbs-cms.md` — 호스트 CMS 실측 기록. 다른 호스트에는 값이 아니라 **측정 항목 5가지**를 재사용

---

## marketplace 2.1.0 — 2026-08-28

### 라이선스 — `UNLICENSED` → `MIT` (플러그인 5종 전부)
포크해서 자기 팀 규약으로 고쳐 쓰라고 만든 저장소인데 라이선스가 사용권을 주지 않고 있어 모순이었습니다. `LICENSE` 파일 추가.
- `css-architecture` `static-site-includes` `scroll-reveal-motion` 1.0.0 → 1.0.1
- `html-layout-conventions` 1.1.0 → 1.1.1

### 변경 — `web-publishing-workflow` 1.0.0 → 1.1.0
기획 쪽에서도 쓰이도록 확장. 4절의 "시안에 담기지 않는 것"(움직임·인터랙션·상태·실제 동작)은 기획서에서 빠지는 항목과 정확히 같은데, 트리거가 퍼블리셔 말투뿐이라 기획서를 쓸 때는 뜨지 않았습니다.
- 트리거 추가: "기획서 쓰는데", "요건 정리", "화면 정의서", "스토리보드", "개발자한테 넘길 문서", "퍼블리셔한테 전달", "빠진 거 없나"
- 4절을 양방향으로: 받는 쪽은 코딩 전에 묻고, 넘기는 쪽은 기획서에 적는다. 넘기기 전 점검용 체크리스트 문장 추가
- 요약표 4행도 양방향으로 수정

---

## marketplace 2.0.0 — 2026-08-28

`ai-plugin-html-layout-conventions` 저장소를 대체하는 새 마켓플레이스. 플러그인 1개에서 5개로 분할하고, 저장소 이름을 마켓플레이스 이름(`frontend-standards`)에 맞췄습니다.

### 신규 — `web-publishing-workflow` 1.0.0
코드를 쓰기 **전** 단계의 규칙. 다중 페이지 작업의 실패는 코드 품질이 아니라 일관성 붕괴라는 전제.
- 레이어 순서: 공통기반 → 페이지구조 → 반응형 → 애니메이션 (페이지 단위 수직 완성 금지)
- 빌드 전 재사용 매핑 제시 후 확인받기
- 시안에 담기지 않는 정보 4종(움직임·인터랙션·상태·실제 동작) 코딩 전 확인
- 다단계 페이지 세트는 템플릿 1개 확정 후 복제
- 임의 진행 금지 / 디자인 유지 · 구조만 변경
- 한 프로젝트 한 도구, 분리할 땐 작업이 아니라 파일 구간으로

### 신규 — `css-architecture` 1.0.0
CSS 파일 구성과 공통 기반 정의.
- 파일 분할은 **페이지가 아니라 컴포넌트 성격** 기준
- 공통 vs 페이지 전용 판단 기준 (2개+ 재사용 / 1회성·충돌위험·무거운 1페이지)
- 로드 순서 `basic → common → sub·main → 페이지 전용`
- `:root` 변수명 표준 — 글자색은 역할 기준, 배경은 층위 기준, `--col-<hex>` 형태 금지
- 페이지 전용 CSS에 태그 글로벌 셀렉터 금지
- `references/basic-css-baseline.md` — 리셋·토큰·폼 기본값 전문 (이전 저장소에서 참조만 되고 실제로는 없던 파일)

### 신규 — `static-site-includes` 1.0.0
빌드툴 없는 정적 사이트의 공통영역 처리.
- `loadInc` 헬퍼와 null 가드 (같은 스크립트를 모든 페이지에 그대로 쓰기 위한 핵심)
- 플레이스홀더 주석 규칙 `<div id="site-header"></div><!-- 경로 -->`
- 삽입 DOM에 붙는 초기화는 반드시 콜백에서 / 중첩 인클루드는 체이닝
- 프래그먼트에 `<script>` 넣으면 실행되지 않음
- 현재 메뉴는 수동 인덱스 대신 URL 경로 매칭
- 프래그먼트 내부 경로는 절대경로 필수 / `file://` 미지원 등 트레이드오프 명시
- `references/loadinc.md` — 구현 전문

### 신규 — `scroll-reveal-motion` 1.0.0
스크롤 등장 모션과 단계 플로우.
- `data-reveal` 그룹 모드(직계 자식 자동 + stagger) / 개별 모드
- **`threshold: 0` 고정** — 올리면 화면보다 긴 요소가 비율 미달로 영영 안 뜸. 타이밍은 `rootMargin` 하단 음수로만
- `prefers-reduced-motion` · `IntersectionObserver` 미지원 시 즉시 표시 (기본 상태가 `opacity: 0` 이라 관찰 실패 = 콘텐츠 소실)
- 1회 실행 후 `unobserve`
- 데스크탑에서 항상 보이는 요소는 해당 구간 리빌 무효화
- `step_flow` 화살표는 `offsetTop` 비교로 같은 줄일 때만, 리사이즈마다 재계산
- `references/implementation.md` — JS·CSS 전문 + 실패 증상별 원인표

### 변경 — `html-layout-conventions` 1.0.1 → 1.1.0
- **추가**: 3절 모디파이어·상태·유틸 — 인덱스형/의미형/조합형 모디파이어, 접두어 없는 공용 상태 클래스(`active`·`open`·`in`·`on`·`done`·`show`·`err`), 유틸(`m_b20`·`txt_center`·`f_between`). 상태 동의어 신설 금지
- **추가**: 변형이 달라도 같은 의미의 자식은 같은 클래스명 (`bd_st1`·`bd_st2` 모두 `bd_tt`)
- **수정**: 존재하지 않는 `references/basic-css-baseline.md` 참조 2곳 제거 → `css-architecture` 스킬로 안내
- **수정**: 라디오·체크박스 설명이 실제 구현과 불일치 (`accent-color` 네이티브 렌더링 → 22px 커스텀 렌더링)
- **수정**: `description`에 "실제로 작성·수정할 때"를 명시해 `web-publishing-workflow`와 트리거 분리
- 마무리 체크리스트 8 → 9항목 (상태 동의어 확인 추가)

---

## 이전 저장소 (`ai-plugin-html-layout-conventions`)

### 1.0.1
- 규약 업데이트

### 1.0.0
- `html-layout-conventions` 플러그인 마켓플레이스 최초 배포
