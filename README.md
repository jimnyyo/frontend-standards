# frontend-standards — Claude 플러그인 마켓플레이스

프론트엔드 퍼블리싱 표준을 Claude(Cowork / Claude Code)에 배포하기 위한 저장소입니다.
저장소 하나가 곧 "마켓플레이스"이고, 팀원은 이 저장소를 한 번만 등록해두면 이후 갱신본을 받아갑니다.

> ⚠️ **이것은 보편 표준이 아니라 특정 팀의 house rule 입니다.**
> 클래스 구분자를 `_`로 강제하거나 CSS를 한 줄로 쓰는 등, 일반적인 생태계 관례(BEM의 `-`, Prettier 기본 포맷)와 다른 결정이 들어 있습니다.
> 그대로 설치하기보다 **포크해서 자기 팀 규약으로 고쳐 쓰는 것**을 전제로 만들었습니다.
> 팀 규약을 스킬로 만드는 구조(트리거 문구 설계, references 분리, 버전 관리 워크플로) 자체가 참고 대상입니다.

## 담고 있는 플러그인

작업의 **시점**에 따라 나뉘어 있습니다. 필요한 것만 골라 설치해도 됩니다.

| 플러그인 | 언제 뜨나 | 내용 |
|---|---|---|
| `web-publishing-workflow` | 코드 쓰기 **전** | 공통기반→페이지구조→반응형→애니메이션 레이어 순서, 코딩 전 재사용 매핑 확인, 시안에 담기지 않는 움직임·인터랙션·상태 확인, 한 프로젝트 한 도구 |
| `html-layout-conventions` | 마크업·CSS를 **쓸 때** | 레이아웃 골격, 언더스코어 네이밍 + 약어 사전, 모디파이어·상태·유틸 클래스, 태그 뎁스 최소화, CSS 한 줄 선언, `:where()` 특이도 제어 |
| `css-architecture` | 프로젝트 **세팅**·CSS 정리 | 컴포넌트 성격 기준 파일 분할, 공통 vs 페이지 전용 판단, 로드 순서, `:root` 변수명 표준, 리셋 기준서 |
| `static-site-includes` | **공통영역** 만들 때 | `loadInc` fetch 인클루드, 플레이스홀더 주석 규칙, 콜백 초기화, null 가드, 현재 메뉴 URL 자동 매칭 |
| `scroll-reveal-motion` | **모션** 붙일 때 | `data-reveal` 그룹·개별, `threshold: 0` 고정 이유, `rootMargin` 타이밍, `prefers-reduced-motion`, 단계 플로우 화살표 |

다섯 개가 서로 겹치지 않게 트리거 문구를 시점 기준으로 갈라놓았습니다. 전부 설치해도 매번 다 로드되지는 않습니다.

> `web-publishing-workflow`는 **넘기는 쪽에서도** 쓸 수 있습니다. 4절의 "시안에 담기지 않는 것"(움직임·인터랙션·상태·실제 동작)이 기획서에서 빠지는 항목과 같아서, 화면 정의서나 요건 정리를 넘기기 전 점검용으로 뜹니다.

## 구조

```
.claude-plugin/marketplace.json          ← 이 저장소가 마켓플레이스임을 알리는 카탈로그
plugins/
  <플러그인명>/
    .claude-plugin/plugin.json           ← 플러그인 매니페스트 (version 여기서 올림)
    skills/<스킬명>/
      SKILL.md                           ← 항상 로드되는 본문
      references/*.md                    ← Claude가 필요할 때만 읽는 참조 문서
```

**SKILL.md 는 스킬이 트리거되면 전문이 컨텍스트에 올라가고, `references/` 는 필요할 때만 읽힙니다.**
그래서 무엇을 어디에 넣을지는 이 기준으로 나눕니다:

| | 위치 |
|---|---|
| 어기면 결과물이 틀리는 **값·규칙** (브레이크포인트, 네이밍, 특이도) | `SKILL.md` |
| 그걸 만들 때만 필요한 **구조·구현** (컴포넌트 마크업 전문, JS 구현) | `references/` |

플러그인을 하나 더 추가하려면 `plugins/` 아래에 같은 모양의 폴더를 만들고 `marketplace.json`의 `plugins` 배열에 항목을 추가하면 됩니다.

---

## 설치 (1회)

**Cowork 데스크톱**

Customize → Plugins → Browse plugins → **Add marketplace** → 아래 주소 입력 → 목록에서 필요한 플러그인 **Install**.

```
jimnyyo/frontend-standards
```

**Claude Code CLI**

```bash
/plugin marketplace add jimnyyo/frontend-standards

/plugin install web-publishing-workflow@frontend-standards
/plugin install html-layout-conventions@frontend-standards
/plugin install css-architecture@frontend-standards
/plugin install static-site-includes@frontend-standards
/plugin install scroll-reveal-motion@frontend-standards
```

설치 후 `/plugin` → **Marketplaces** 탭에서 `frontend-standards`를 선택해 **auto-update를 켜 두세요.** 서드파티 마켓플레이스는 자동 갱신이 기본으로 꺼져 있습니다.

## 규약을 고쳤을 때 (관리자)

```bash
# 1) SKILL.md 또는 references/*.md 수정
# 2) 버전 올리기 — 두 곳 다
#    plugins/<플러그인명>/.claude-plugin/plugin.json  의 "version"
#    .claude-plugin/marketplace.json 의 해당 plugin 항목 "version"
# 3) CHANGELOG.md 에 한 줄 추가
git add -A && git commit -m "규약 수정: <무엇을 바꿨는지>" && git push
```

버전을 올리지 않으면 파일이 바뀌어도 클라이언트가 갱신을 인식하지 못할 수 있습니다. **버전 올리는 것을 커밋과 한 세트로 취급하세요.**

## 갱신본이 내려오는 방식

- auto-update를 켜 두면 세션 시작 후 백그라운드에서 마켓플레이스를 새로 읽어옵니다(최대 10분 내 무작위 지연). **실행 중인 세션은 시작 시점 버전을 계속 쓰므로**, 반영은 다음 실행 때 또는 `/reload-plugins` 후에 됩니다.
- 지금 당장 받고 싶으면:
  ```bash
  /plugin marketplace update frontend-standards
  /reload-plugins
  ```
  Cowork에서는 마켓플레이스 목록의 **Update** 버튼을 누르면 됩니다.
- `/plugin install`은 항상 마켓플레이스를 먼저 새로 읽으므로, 재설치해도 최신본을 받습니다.

## 프로젝트별로 달라지는 값

규약 본문에는 특정 조직·프로젝트 정보를 넣지 않았습니다. 아래는 프로젝트마다 다르므로, 규약은 "프로젝트에서 이미 쓰는 값을 따르고 분명하지 않으면 물어볼 것"으로 되어 있습니다.

| 값 | 어디서 정하나 |
|---|---|
| 이미지·인클루드 절대경로 프리픽스 (`/<프로젝트루트>/...`) | 프로젝트 코드에서 이미 쓰는 경로 |
| `basic.css`의 CSS 변수 실제 색상값 | 프로젝트의 실제 `basic.css` (변수 *이름* 표준은 `css-architecture`에 있음) |
| 브레이크포인트 숫자 | 프로젝트 파운데이션에서 확정 (예시는 1280 / 768 / 480) |

특정 프로젝트에 고정하고 싶으면 해당 SKILL.md에 실제 값을 적어 넣으세요. 다만 이 저장소는 공개이므로, 외부에 알려지면 곤란한 내부 경로는 넣지 마세요.

## 프로젝트 CLAUDE.md 와의 관계

**같은 규칙을 두 곳에 두지 마세요.** 스킬로 옮긴 내용은 프로젝트 `CLAUDE.md` 에서 지우고 한 줄 참조로 바꿉니다.

```markdown
> HTML/CSS 작성 규칙은 `html-layout-conventions` 스킬 참조.
> CSS 파일 구성·토큰은 `css-architecture` 스킬 참조.
```

`CLAUDE.md` 에 **남겨야 하는 것**은 스킬로 일반화할 수 없는 프로젝트 고유 사실입니다 — 폴더 구조, 실제 경로 프리픽스, 컴포넌트 인벤토리, 작업 현황, 배포 방법.

## 라이선스

[MIT](LICENSE) — 포크·수정·재배포 자유입니다. 애초에 그대로 쓰기보다 **자기 팀 규약으로 고쳐 쓰라고** 만든 저장소입니다.
