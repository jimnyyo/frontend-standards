# CHANGELOG

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
