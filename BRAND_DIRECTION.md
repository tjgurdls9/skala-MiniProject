# Jobpill Brand Direction

## Brand idea

**Job + 수필(隨筆)**

Jobpill은 채용 정보를 더 쌓는 서비스가 아니라, 사용자의 커리어 초고를 현직자 기준으로 읽고 부족한 경험을 찾아 다음 프로젝트로 퇴고하게 하는 서비스다.

- 핵심 동사: 읽다 → 대조하다 → 고쳐 쓰다
- 핵심 소재: 원고지, 문장, 여백, 문단, 책갈피, 마침표
- 제외 소재: 알약/캡슐, 의료, 나침반, 상승 화살표, 반짝이, AI 뇌

## Typography

- Typeface: Pretendard
- Color: 로고·워드마크는 순수 블랙 `#000000`. 본문 잉크는 토큰 `--ink: #0B0D0C`를 쓴다.
- Display: 800 / 64px / 1.12 / -0.045em
- Page title: 800 / 40px / 1.15 / -0.04em
- Section title: 800 / 22px / 1.35 / -0.03em
- Panel · Modal title: 800 / 22px / 1.35 / -0.03em (Section title과 같은 급)
- Card title: 800 / 18px / -0.03em
- Lead: 500 / 18px / 1.70 / -0.015em
- Body: 400 / 16px / 1.70 / -0.01em
- UI label: 700 / 13px / 1.40 / 0
- Caption: 500 / 12px / 1.50 / 0
- Wordmark: 700 / line-height 1 / -0.045em

원칙: 굵기는 400, 500, 700, 800만 사용한다. 제목은 짧고 단단하게, 설명문은 긴 호흡으로 둔다. 브랜드 표현에서는 회색이나 포인트 컬러 없이 블랙만 사용한다.

## Logo direction

**확정안: `jp` 문자 모노그램 + `Jobpill` 워드마크 (2026.09.04 결정)**

`j`와 `p`를 한 획으로 묶은 모노그램 왼쪽에, Pretendard 계열 700 워드마크를 오른쪽에 둔 가로 조합형이다.
`j`의 점만 네온 그린 `#38FF9C`로 남기고 나머지는 순수 블랙으로 간다 — 색은 한 점에만 쓴다.

- 심볼과 워드마크를 붙인 **락업(lockup)**이 기본형이다. 헤더·푸터·로그인 등 로고가 들어가는 모든 자리에 락업을 쓴다.
- **심볼 단독형**은 파비콘처럼 정사각 최소 크기에서만 쓴다.
- 어두운 면에서는 검정 잉크를 `#EDEFED`로 반전한 버전을 쓰고, **네온 그린 점은 반전하지 않는다.**

에셋 (`src/assets/logo/`):

| 파일 | 용도 |
|---|---|
| `jobpill-lockup.png` | 밝은 면 기본형 (헤더) |
| `jobpill-lockup-dark.png` | 어두운 면 (푸터) |
| `jobpill-mark.png` | 심볼 단독, 밝은 면 |
| `jobpill-mark-dark.png` | 심볼 단독, 어두운 면 |
| `favicon-32.png` / `favicon-180.png` | 브라우저 탭 · 홈 화면 아이콘 |

컴포넌트는 `src/components/Logo.vue` 하나이며 `size`(높이 기준) · `dark` · `markOnly` props로 위 4종을 고른다.
로고를 바꿔야 할 때는 이 컴포넌트와 위 에셋만 교체하면 전 화면에 반영된다.

> 아래 "Non-letter explorations"와 "Single-style comparison"은 **채택되지 않은 탐색 기록**이다. 결과적으로 글자를 쓰지 않는 독립 심볼 방향은 접고, `jp` 모노그램으로 돌아왔다.

- 미니멀리즘: 작은 크기에서도 남는 단순한 실루엣
- 퓨처리스틱: 기하 형태와 정밀한 비례
- 글래스모피즘: 전체 효과가 아니라 겹침과 깊이를 설명하는 한 레이어
- 스위스 디자인: 명확한 그리드와 비대칭 균형
- Apple 디자인: clarity, deference, depth
- Toss 디자인: 친근한 곡률과 즉시 읽히는 덩어리감
- Palette: neon green `#38FF9C`, black `#000000`, white `#FFFFFF`

### Non-letter explorations

1. **Overlap** — 개인 프로필과 현직자 기준이 겹쳐 초록색 핵심이 드러난다.
2. **Path** — 기준 프레임을 통과하며 정렬되는 하나의 경로다.
3. **Coordinate** — 여러 기준 가운데 사용자의 현재 좌표를 찾는다.
4. **Loop** — 진단, 실행, 재진단의 반복을 하나의 접힌 띠로 표현한다.
5. **Focus** — 많은 정보 안에서 보완할 핵심 경험 하나를 찾는다.

추천 순위는 **1안 → 3안 → 2안**이다.

### Single-style comparison

동일한 의미인 “개인 프로필과 현직자 기준이 만나 다음 핵심이 드러난다”를 유지한 채, 조형 언어를 섞지 않고 다음 여섯 스타일로 각각 분리했다.

1. Minimalism — 완전 평면, 세 도형, 겹침만으로 의미 전달
2. Futuristic — 날카롭고 공학적인 전진형 조형
3. Glassmorphism — 투명 레이어와 굴절로 드러나는 교집합
4. Swiss — 수학적 그리드와 비대칭 평면 구성
5. Apple — 조용한 곡면, 정밀한 광학 균형, 제한된 깊이
6. Toss — 친근한 매트 볼륨과 즉시 읽히는 관계

원본은 `src/assets/logo-concepts/single-style/`에 저장한다.

## 문자 기반 탐색안 (초기)

1. **원고지 밖의 한 줄** — 한 칸을 넘어가는 문장으로 기존 프레임을 확장한다.
2. **두 번째 문장** — 두 줄과 `j`를 결합해 초고와 수정본을 표현한다.
3. **빠진 문장** — 문단의 여백이 `J`가 되어 부족한 경험을 찾는 서비스를 표현한다.
4. **이어 쓰는 책갈피** — 페이지와 `p` 형태의 책갈피로 계속 갱신되는 커리어를 표현한다.
5. **JP 마침표** — `j`와 `p`를 한 획으로 묶고 마침표를 더해 끝과 새 시작을 표현한다. ← **이 방향이 최종안이 됐다.** 마침표는 네온 그린 점으로 살아남았다.

1~4번은 채택하지 않았다.

## Generated exploration assets

- `src/assets/logo-concepts/jobpill-logo-01.png`
- `src/assets/logo-concepts/jobpill-logo-02.png`
- `src/assets/logo-concepts/jobpill-logo-03.png`
- `src/assets/logo-concepts/jobpill-logo-04.png`
- `src/assets/logo-concepts/jobpill-logo-05.png`
- `src/assets/logo-concepts/no-letter/jobpill-symbol-01-overlap.png`
- `src/assets/logo-concepts/no-letter/jobpill-symbol-02-path.png`
- `src/assets/logo-concepts/no-letter/jobpill-symbol-03-coordinate.png`
- `src/assets/logo-concepts/no-letter/jobpill-symbol-04-loop.png`
- `src/assets/logo-concepts/no-letter/jobpill-symbol-05-focus.png`

생성 이미지는 심볼 방향을 비교하기 위한 탐색안이다. 최종안은 선택된 심볼을 벡터로 다시 그리고, 워드마크를 실제 Pretendard 700으로 조판한다.

## Generation prompt set

공통 조건은 `Jobpill` 정확 표기, Job + essay 콘셉트, Pretendard 계열 700 워드마크, 순수 블랙 단색, 투명 배경, 플랫 벡터 친화형, 24px 가독성이다. 각 안에는 순서대로 원고지와 연장선, 두 문장과 j, 문단 여백과 J, 페이지·책갈피와 p, 공유 획 JP와 마침표를 지정했다. 알약·의료·나침반·상승 화살표·반짝이·AI 뇌·마스코트는 모두 제외했다.
