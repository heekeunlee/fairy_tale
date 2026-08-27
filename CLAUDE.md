# fairy_tale — 창작동화 제작 파이프라인

이 프로젝트는 여러 역할 에이전트가 협업해 창작동화를 기획-집필-심사-삽화-홍보하는 워크플로우입니다.
핵심 3개 에이전트(기획자·작가·심사위원)로 시작해 MVP를 검증한 뒤, 북디자이너·트렌드 리서처·
마케터·총무·고문까지 8개 역할로 확장했습니다.

## 현재 구성 (8개 역할)

| 역할 | 에이전트 | 정의 파일 | 트리거 |
|---|---|---|---|
| 기획자 | planner | `.claude/agents/planner.md` | `/plan <주제>` |
| 작가 | writer | `.claude/agents/writer.md` | `/write [기획서 경로]` |
| 심사위원 | critic | `.claude/agents/critic.md` | `/review [초고 경로]` |
| 북디자이너 | designer | `.claude/agents/designer.md` | `/illustrate [작품]` |
| 트렌드 리서처 | researcher | `.claude/agents/researcher.md` | `/research [주제]` |
| 마케터 | marketer | `.claude/agents/marketer.md` | `/market [작품]` |
| 총무 | admin | `.claude/agents/admin.md` | `/audit` |
| 고문 (아동문학가) | advisor | `.claude/agents/advisor.md` | `/consult [작품]` |

### 트렌드 리서처의 두 번째 업무: 저작권·유사성 리스크 점검
매주 1회, 완성작이 기존/출간예정 작품과 유사해 법적 리스크가 없는지 점검하고 주간보고를
작성한다 (`research/{날짜}-주간보고.md`). **주의: 이건 저절로 매주 실행되지 않는다.** 이
세션은 사용자가 말을 걸어야 움직이므로, 실제로 매주 실행되게 하려면 (a) 매주 사용자가 직접
`/research` 로 리스크 점검을 요청하거나, (b) `schedule` 스킬로 클라우드 예약 루틴을 만들어야
한다 — 후자는 `fairy_tale`이 GitHub 저장소여야 접근 가능하므로 아직 설정되어 있지 않다.

### 고문의 역할
심사위원(critic)이 "기획대로 잘 만들었는가"를 본다면, 고문은 "이게 좋은 문학이고 아이에게
좋은가"를 별도 관점(문학적 완성도·아동 심리 발달·인문학적 함의)에서 검토한다. 작품이 새로
통과되거나 개정될 때마다 실행하는 게 원칙이지만, 이 역시 자동 트리거가 아니라 사람이
`/consult`로 불러야 한다.

이 역할들은 `fairy_tale_office` 저장소(오피스 화면)의 크루 명단과 1:1로 대응합니다.
사람 이름/사진이 붙어 있는 건 오피스 화면 연출이고, 실제로는 전부 Claude Code 서브에이전트입니다.

## 워크플로우

1. (선택) `/research`로 트렌드를 조사해 소재 후보를 얻는다.
2. `/plan 용감한 달팽이 이야기` 처럼 주제를 주고 기획서를 만든다.
3. 기획서를 검토(필요시 직접 수정)한 뒤 `/write`로 초고를 집필한다.
4. `/review`로 초고를 심사받는다.
5. 판정이 "수정 필요"면 `/write`를 다시 실행해 피드백을 반영한 v2를 만들고, `/review`를 반복한다.
6. "통과"가 나오면 `/consult`로 고문 검토를 받고, `/illustrate`로 삽화를, `/market`으로 홍보 카피를 만들어 `fairy_tale_office`에 반영한다.
7. 가끔 `/audit`으로 전체 정합성을, `/research`로 트렌드·저작권 리스크를 점검한다.

각 단계 결과는 사람이 확인하고 다음 단계로 넘길지 결정하는 반자동(semi-manual) 방식입니다.
완전 자동화(크론 기반 무인 파이프라인)는 의도적으로 아직 도입하지 않았습니다 — 초기에는
품질 검증 없이 자동 진행되면 오류가 누적되기 쉽기 때문입니다.

## 파일 명명 규칙

- 기획서: `planning/{YYYY-MM-DD}-{제목슬러그}.md`
- 초고: `drafts/{제목슬러그}-v{버전}.md`
- 심사: `reviews/{제목슬러그}-v{버전}-review.md`

같은 이야기의 기획서/초고/리뷰는 제목슬러그로 서로 연결됩니다.

- 트렌드 조사: `research/{YYYY-MM-DD}-{주제}.md`
- 저작권 주간보고: `research/{YYYY-MM-DD}-주간보고.md`
- 마케팅 카피: `marketing/{제목슬러그}.md`
- 총무 점검: `admin/{YYYY-MM-DD}-점검.md`
- 고문 검토: `advisory/{제목슬러그}-v{버전}-advisory.md`

## fairy_tale_office 연동

완성작과 관련 산출물은 별도 저장소 `fairy_tale_office`(오피스 화면, GitHub Pages)에 반영됩니다.
- 통과작 → `site/stories/{제목슬러그}.html`, `site/index.html`의 서재 목록
- 삽화 → 같은 리더 페이지에 SVG로 통합 (designer 담당)
- 크루 스탯(기획서/초고/심사 건수)은 아직 수동 갱신 — admin이 정합성만 점검하고, 실제 자동 동기화는
  두 저장소를 연결하는 스크립트가 생기면 그때 자동화할 것.

새 에이전트를 추가할 때는 `.claude/agents/`에 정의 파일을, 필요하면 `.claude/commands/`에
트리거용 슬래시 커맨드를 만들고 이 문서의 구성 표를 갱신할 것.
