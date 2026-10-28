# Daily Curation Blog

날짜별 기술 뉴스 큐레이션 JSON을 읽어 아카이브와 일자별 게시물 페이지를 만드는 Astro 정적 사이트입니다.

## 주요 기능

- **아카이브 페이지** (`/`): `public/data/`의 JSON 파일을 최신 날짜순으로 나열하고, 게시물마다 요약과 기사·트렌드·액션 개수를 보여줍니다.
- **일자별 게시물 페이지** (`/posts/YYYYMMDD`): JSON 파일 하나당 페이지 하나를 빌드 시점에 생성합니다(`getStaticPaths`). 표시하는 섹션은 다음과 같습니다.
  - 요약 배너
  - 🔥 트렌드: 영향도(`high` / `medium`)와 출처 기사 제목
  - 💡 인사이트
  - ⚡ 액션 아이템: 우선순위(`high` / `medium` / `low`), 소요 시간, 관련 트렌드
  - 📊 Business Impact: 기회, 위협, 권고사항
  - 📚 원문 기사 목록: 출처 태그, 제목, 설명(HTML 태그를 제거하고 150자로 자름), 원문 링크
- 데이터가 없는 섹션은 렌더링하지 않습니다. 게시물이 하나도 없으면 아카이브에 빈 상태 안내가 나옵니다.
- 다크 테마 UI(`src/styles/global.css`)를 사용하며 페이지 언어는 `ko`입니다.

## 기술 스택

- [Astro](https://astro.build) 6 (`output: 'static'`)
- TypeScript (`astro/tsconfigs/strict`)
- Node.js `>=22.12.0` (`package.json`의 `engines`)
- Vercel 배포 설정(`vercel.json`)

## 폴더 구조

```text
.
├── astro.config.mjs        # Astro 설정 (정적 출력)
├── vercel.json             # Vercel 빌드 설정
├── package.json
├── tsconfig.json
├── public/
│   ├── data/               # 게시물 데이터 (YYYYMMDD.json), 빌드 시 읽음
│   ├── favicon.ico
│   └── favicon.svg
└── src/
    ├── data/               # public/data와 같은 JSON 사본 (현재 코드에서 읽지 않음)
    ├── layouts/
    │   └── BlogLayout.astro  # 공통 헤더, 푸터, 메타 태그
    ├── pages/
    │   ├── index.astro       # 아카이브 목록
    │   └── posts/
    │       └── [date].astro  # 일자별 게시물
    └── styles/
        └── global.css
```

## 데이터 형식

`public/data/YYYYMMDD.json` 파일 하나가 게시물 하나가 됩니다. 파일명(확장자 제외)이 날짜이자 URL 경로입니다. 페이지에서 사용하는 최상위 필드는 다음과 같습니다.

| 필드 | 내용 |
| --- | --- |
| `date`, `generated_at` | 날짜, 생성 시각 |
| `summary` | 하루치 요약 문장 |
| `trends[]` | `title`, `description`, `sources[]`, `impact` |
| `insights[]` | `title`, `content`, `trend_ref` |
| `action_items[]` | `title`, `description`, `priority`, `effort`, `trend_ref` |
| `business_impact` | `opportunities[]`, `threats[]`, `recommendations[]` (문자열 배열) |
| `articles[]` | `title`, `link`, `description`, `source` 등 |

> `business_impact`의 항목은 `set:html`로 렌더링되므로 HTML이 그대로 들어갑니다. 신뢰할 수 있는 데이터만 넣어야 합니다.

JSON을 생성하거나 수집하는 스크립트와 스케줄 워크플로는 이 저장소에 없습니다. 푸터의 "매일 오후 6시 자동 업데이트" 문구는 외부에서 데이터 파일을 추가하는 방식을 전제로 합니다.

## 실행 및 빌드

```sh
npm install
npm run dev       # 개발 서버
npm run build     # ./dist 에 정적 사이트 생성
npm run preview   # 빌드 결과 미리보기
```

새 게시물을 추가하려면 `public/data/`에 `YYYYMMDD.json`을 넣고 다시 빌드합니다. 데이터는 빌드 시점에만 읽으므로, 파일을 추가한 뒤에는 재빌드나 재배포가 필요합니다.

## 배포

`vercel.json`은 Vercel용 설정입니다(`framework: astro`, `npm install` 후 `npm run build`, 출력 디렉터리 `dist`). `@astrojs/vercel` 패키지가 의존성에 있지만 `astro.config.mjs`에서 어댑터로 설정하지 않았기 때문에, 사이트는 순수 정적 파일로 빌드됩니다.

## 환경 변수

필요한 환경 변수는 없습니다.
