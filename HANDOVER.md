# 한영 기술용어집 (Tech Glossary) 인수인계 문서

> **최종 업데이트:** 2026-02-24
> **대상:** Confluence 위키페이지 관리 담당자
> **작성 목적:** 기술용어집 시스템의 구조 이해 및 유지보수 방법 안내

---

## 목차

1. [시스템 개요](#1-시스템-개요)
2. [전체 구조도](#2-전체-구조도)
3. [파일 구성](#3-파일-구성)
4. [index.html 구조 상세](#4-indexhtml-구조-상세)
5. [JSON 데이터 파일 규격](#5-json-데이터-파일-규격)
6. [Confluence 위키 배포 방법](#6-confluence-위키-배포-방법)
7. [용어 추가/수정/삭제 방법](#7-용어-추가수정삭제-방법)
8. [새로운 공종(Discipline) 추가 방법](#8-새로운-공종discipline-추가-방법)
9. [자주 발생하는 문제와 해결 방법](#9-자주-발생하는-문제와-해결-방법)
10. [주의사항](#10-주의사항)

---

## 1. 시스템 개요

### 이 시스템은 무엇인가?

SAMOO 하이테크 1본부에서 해외 프로젝트 통역/번역 업무를 위해 사용하는 **한영 기술용어집 웹 애플리케이션**입니다. Confluence 위키페이지에 올려서 사내 누구나 브라우저로 접속하여 사용할 수 있습니다.

### 핵심 동작 원리

```
[Confluence 위키페이지]
        │
        ├── index.html (HTML 매크로에 삽입된 웹 애플리케이션)
        │
        └── glossary-data/ (첨부파일로 업로드된 JSON 파일들)
                ├── general.json
                ├── architecture.json
                ├── civil-1.json ~ civil-4.json
                └── ... (총 14개 파일)

① 사용자가 Confluence 페이지 접속
② index.html이 Confluence REST API를 통해 같은 페이지의 첨부파일(JSON) 목록을 조회
③ 각 JSON 파일을 다운로드하여 브라우저 메모리에 로드
④ 사용자에게 검색, 필터, 페이지네이션 기능이 포함된 용어 테이블 표시
```

### 주요 기능

| 기능 | 설명 |
|------|------|
| 용어 검색 | 영어, 한국어, 설명 필드를 대상으로 실시간 검색 (오타 허용하는 퍼지 검색 포함) |
| 공종별 보기 | 11개 공종(General, Architecture, Civil 등)별 필터링 |
| 전체 보기 | 모든 공종의 용어를 한꺼번에 표시 |
| 페이지네이션 | 50개 단위로 페이지 나눔 |
| 반응형 디자인 | 모바일에서도 사용 가능 |

---

## 2. 전체 구조도

```
tech-glossary/
│
├── index.html                          ← 메인 애플리케이션 (HTML + CSS + JS 올인원)
├── style.html                          ← 스타일 참고용 파일 (실제 사용 안 함)
├── HANDOVER.md                         ← 본 인수인계 문서
│
└── glossary-data/                      ← 용어 데이터 (JSON 파일들)
    ├── general.json                    ← 일반 (약 500개 용어)
    ├── architecture.json               ← 건축
    ├── civil-1.json                    ← 토목 (용량이 커서 4분할)
    ├── civil-2.json                    ← 토목
    ├── civil-3.json                    ← 토목
    ├── civil-4.json                    ← 토목
    ├── structure.json                  ← 구조
    ├── fire-protection.json            ← 소방
    ├── piping.json                     ← 배관
    ├── electrical.json                 ← 전기
    ├── instrument-and-control.json     ← 제어
    ├── hvac.json                       ← 공조
    ├── bim.json                        ← BIM
    └── cell.json                       ← 배터리
```

---

## 3. 파일 구성

### 3.1 index.html (메인 파일)

총 927줄의 단일 HTML 파일로, CSS 스타일과 JavaScript가 모두 내장되어 있습니다.

| 영역 | 줄 번호 (대략) | 내용 |
|------|---------------|------|
| CSS 스타일 | 6~128행 | 전체 UI 디자인, 반응형 레이아웃, Confluence 호환 스타일 |
| HTML 본문 | 131~190행 | 헤더, 검색창, 보기 컨트롤, 테이블, 페이지네이션 UI |
| JavaScript | 195~927행 | 데이터 로딩, 검색, 필터링, 렌더링 로직 |

### 3.2 JSON 데이터 파일 (glossary-data/)

모든 용어 데이터가 들어있는 파일들입니다. Confluence 페이지의 **첨부파일**로 업로드되어 사용됩니다.

| 파일명 | 공종 | 대략적 용어 수 |
|--------|------|--------------|
| general.json | 일반 | ~500 |
| architecture.json | 건축 | ~660 |
| civil-1.json ~ civil-4.json | 토목 | ~11,300 (4개 파일 합계) |
| structure.json | 구조 | ~130 |
| fire-protection.json | 소방 | ~700 |
| piping.json | 배관 | ~120 |
| electrical.json | 전기 | ~170 |
| hvac.json | 공조 | ~120 |
| instrument-and-control.json | 제어 | ~30 |
| bim.json | BIM | ~140 |
| cell.json | 배터리 | ~490 |

---

## 4. index.html 구조 상세

### 4.1 HTML 구조

```
<div class="tech-glossary">           ← 최상위 컨테이너
  <link ... font-awesome ...>         ← 아이콘 라이브러리 (외부 CDN)
  <style> ... </style>                ← 내장 CSS

  ┌─ Header ──────────────────────────┐
  │ 제목: English-Korean Technical    │
  │       Glossary / 한영 기술용어집   │
  └───────────────────────────────────┘

  ┌─ Disclaimer ──────────────────────┐
  │ 안내사항 및 문의처 정보              │
  └───────────────────────────────────┘

  ┌─ Search ──────────────────────────┐
  │ [검색 입력창]                      │
  └───────────────────────────────────┘

  ┌─ Controls ────────────────────────┐
  │ [공종별 보기] [전체 보기]           │
  └───────────────────────────────────┘

  ┌─ Discipline Shortcuts ────────────┐
  │ 동적으로 생성되는 공종 필터 버튼     │
  └───────────────────────────────────┘

  ┌─ Table ───────────────────────────┐
  │ 구분 | EN | KR | 설명              │
  │ (동적으로 데이터 렌더링)             │
  └───────────────────────────────────┘

  ┌─ Pagination ──────────────────────┐
  │ « ‹ [1] [2] [3] ... › »          │
  └───────────────────────────────────┘

  <script> ... </script>              ← 내장 JavaScript
</div>
```

### 4.2 JavaScript 핵심 로직

#### Confluence REST API 연동 (310~354행)

```javascript
// Confluence 페이지 ID (★ 이 값이 Confluence 페이지를 식별합니다)
const PAGE_ID = '56625515';

// 1단계: REST API로 첨부파일 목록 조회
fetch('/rest/api/content/' + PAGE_ID + '/child/attachment?expand=download&limit=50')

// 2단계: 각 JSON 파일의 다운로드 경로 획득
// 3단계: 캐시 방지용 타임스탬프를 붙여 파일 로드
fetch(downloadPath + '?t=' + Date.now())
```

> **중요:** `PAGE_ID`는 현재 `56625515`로 설정되어 있습니다. 위키페이지가 변경되면 이 값도 반드시 수정해야 합니다.

#### 공종-파일 매핑 (296~308행)

```javascript
const jsonFiles = {
  'General': 'general.json',
  'Architecture': 'architecture.json',
  'Civil': ['civil-1.json', 'civil-2.json', 'civil-3.json', 'civil-4.json'],
  'Structure': 'structure.json',
  'Fire Protection': 'fire-protection.json',
  'Piping': 'piping.json',
  'Electrical': 'electrical.json',
  'Instrument & Control': 'instrument-and-control.json',
  'HVAC': 'hvac.json',
  'BIM': 'bim.json',
  'Cell': 'cell.json'
};
```

> **참고:** Civil 공종은 데이터가 많아 4개 파일로 분할되어 있습니다. 새 파일을 추가하려면 여기에도 등록해야 합니다.

#### 공종 정보 (356~368행)

각 공종의 약어, 한글 이름, 아이콘이 정의되어 있습니다:

```javascript
const disciplineMap = {
  'General':              { abbreviation: 'Gen',  koreanName: '일반',   icon: 'fa-solid fa-file-lines' },
  'Architecture':         { abbreviation: 'Arch', koreanName: '건축',   icon: 'fa-solid fa-building' },
  'Civil':                { abbreviation: 'Civ',  koreanName: '토목',   icon: 'fa-solid fa-road' },
  'Structure':            { abbreviation: 'Str',  koreanName: '구조',   icon: 'fa-solid fa-cubes' },
  'Fire Protection':      { abbreviation: 'FP',   koreanName: '소방',   icon: 'fa-solid fa-fire-extinguisher' },
  'Piping':               { abbreviation: 'Pip',  koreanName: '배관',   icon: 'fa-solid fa-faucet-drip' },
  'Electrical':           { abbreviation: 'Elec', koreanName: '전기',   icon: 'fa-solid fa-bolt' },
  'Instrument & Control': { abbreviation: 'I&C',  koreanName: '제어',   icon: 'fa-solid fa-microchip' },
  'HVAC':                 { abbreviation: 'HVAC', koreanName: '공조',   icon: 'fa-solid fa-fan' },
  'BIM':                  { abbreviation: 'BIM',  koreanName: 'BIM',    icon: 'fa-solid fa-cube' },
  'Cell':                 { abbreviation: 'Cell', koreanName: '배터리', icon: 'fa-solid fa-car-battery' }
};
```

---

## 5. JSON 데이터 파일 규격

### 5.1 기본 구조

모든 JSON 파일은 **배열(Array)** 형태이며, 각 요소는 다음 필드를 가집니다:

```json
[
  {
    "en": "영어 용어 (필수)",
    "kr": "한국어 번역 (필수)",
    "description": "부가 설명 (선택, 빈 문자열 가능)",
    "discipline": "공종명 (필수, 영문)"
  }
]
```

### 5.2 실제 예시

```json
[
  {
    "en": "Blind Flange",
    "kr": "맹판 플랜지, 맹플랜지",
    "description": "플랜지 말단부에 체결하여 내부·외부 차단",
    "discipline": "Piping"
  },
  {
    "en": "Black Steel Pipe",
    "kr": "흑강관",
    "description": "",
    "discipline": "Piping"
  }
]
```

### 5.3 필드 상세

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| `en` | string | O | 영어 용어. 첫 글자는 자동으로 대문자 변환됨 |
| `kr` | string | O | 한국어 번역. 여러 번역이 있으면 쉼표(,)로 구분 |
| `description` | string | O | 부가 설명. 없으면 빈 문자열 `""` |
| `discipline` | string | O | 공종명. `jsonFiles` 매핑의 키 값과 **정확히** 일치해야 함 |

### 5.4 discipline 필드 허용 값

다음 값 중 하나를 **정확히** 사용해야 합니다 (대소문자 구분):

- `General`
- `Architecture`
- `Civil`
- `Structure`
- `Fire Protection`
- `Piping`
- `Electrical`
- `Instrument & Control`
- `HVAC`
- `BIM`
- `Cell`

---

## 6. Confluence 위키 배포 방법

### 6.1 전체 작업 흐름

```
[1단계] JSON 파일 편집 (용어 추가/수정/삭제)
    ↓
[2단계] JSON 유효성 검증
    ↓
[3단계] Confluence 페이지에 JSON 파일 첨부파일로 업로드 (기존 파일 교체)
    ↓
[4단계] (필요 시) index.html 수정 후 HTML 매크로 업데이트
    ↓
[5단계] 페이지 새로고침하여 정상 동작 확인
```

### 6.2 JSON 파일 업로드 (가장 빈번한 작업)

용어를 추가/수정/삭제한 후 JSON 파일만 교체하면 됩니다.

#### 순서:

1. **Confluence 페이지 접속**
   - 기술용어집이 게시된 Confluence 페이지로 이동

2. **첨부파일 관리 열기**
   - 페이지 우측 상단 `⋯` (더보기) 메뉴 클릭
   - `첨부파일` (Attachments) 선택

3. **기존 파일 교체 업로드**
   - `파일 업로드` (Upload file) 클릭
   - 수정된 JSON 파일을 선택 (파일명이 기존 파일과 **동일**해야 함)
   - 동일한 이름의 파일이 이미 있으면 Confluence가 자동으로 **새 버전**으로 교체

4. **확인**
   - 페이지로 돌아가 새로고침 (Ctrl+Shift+R 또는 Cmd+Shift+R)
   - 캐시 방지 로직이 내장되어 있으므로 일반 새로고침으로도 대부분 반영됨

### 6.3 index.html 수정 (드문 작업)

HTML 코드 자체를 수정해야 할 경우:

1. **Confluence 페이지 편집 모드 진입**
   - 페이지 우측 상단 `편집` (Edit) 클릭

2. **HTML 매크로 찾기**
   - 페이지 본문에서 `HTML` 매크로 블록을 찾아 클릭
   - 매크로 편집 모드로 진입

3. **HTML 코드 수정**
   - index.html의 전체 내용을 복사하여 매크로 본문에 붙여넣기
   - 또는 필요한 부분만 수정

4. **저장**
   - 매크로 저장 → 페이지 저장(게시)

> **주의:** Confluence의 HTML 매크로는 보안 정책에 따라 일부 HTML/JS가 제한될 수 있습니다. 현재 시스템은 이러한 제한을 고려하여 작성되어 있으므로, 구조를 크게 변경할 때는 주의가 필요합니다.

---

## 7. 용어 추가/수정/삭제 방법

### 7.1 용어 추가

1. 해당 공종의 JSON 파일을 텍스트 에디터로 엽니다
2. 배열 내부에 새 용어 객체를 추가합니다:

```json
  {
    "en": "New Term",
    "kr": "새 용어",
    "description": "설명을 여기에 입력",
    "discipline": "General"
  }
```

3. **주의:** 마지막 항목이 아닌 경우, 객체 뒤에 쉼표(`,`)가 있어야 합니다
4. JSON 유효성 검증 후 Confluence에 업로드

### 7.2 용어 수정

1. JSON 파일에서 해당 용어를 검색 (Ctrl+F)
2. `en`, `kr`, `description` 값을 수정
3. JSON 유효성 검증 후 Confluence에 업로드

### 7.3 용어 삭제

1. JSON 파일에서 해당 용어 객체 전체를 삭제
2. 삭제 후 쉼표(`,`) 정리 (배열의 마지막 요소 뒤에 쉼표가 없어야 함)
3. JSON 유효성 검증 후 Confluence에 업로드

### 7.4 JSON 유효성 검증 (매우 중요)

JSON 형식이 잘못되면 해당 공종의 데이터가 **전혀 로드되지 않습니다**.

#### 온라인 검증 도구:
- https://jsonlint.com/ — JSON 붙여넣기 후 "Validate JSON" 클릭
- https://jsonformatter.curiousconcept.com/ — 포맷팅 및 검증 동시 가능

#### 자주 하는 실수:
```
잘못된 예 ✗                           올바른 예 ✓
─────────────────────────────────────────────────
{ "en": "Term", }                     { "en": "Term" }
  (마지막에 불필요한 쉼표)                 (쉼표 없음)

{ "en": "Term"                        { "en": "Term",
  "kr": "용어" }                        "kr": "용어" }
  (쉼표 누락)                            (쉼표 있음)

{ en: "Term" }                        { "en": "Term" }
  (키에 따옴표 없음)                      (따옴표 필수)

{ "en": "He said "hello"" }          { "en": "He said \"hello\"" }
  (따옴표 이스케이프 안 함)                (백슬래시로 이스케이프)
```

---

## 8. 새로운 공종(Discipline) 추가 방법

새로운 공종을 추가하려면 index.html의 **3곳**을 수정해야 합니다.

### 8.1 단계별 절차

#### Step 1: JSON 파일 생성

새 공종의 JSON 파일을 만듭니다 (예: `mechanical.json`):

```json
[
  {
    "en": "Bearing",
    "kr": "베어링",
    "description": "회전축을 지지하는 기계 요소",
    "discipline": "Mechanical"
  }
]
```

#### Step 2: index.html — `jsonFiles` 매핑에 추가 (296~308행 부근)

```javascript
const jsonFiles = {
  // ... 기존 항목들 ...
  'Cell': 'cell.json',
  'Mechanical': 'mechanical.json'    // ← 추가
};
```

#### Step 3: index.html — `disciplineMap`에 추가 (356~368행 부근)

```javascript
const disciplineMap = {
  // ... 기존 항목들 ...
  'Cell': { abbreviation: 'Cell', koreanName: '배터리', icon: 'fa-solid fa-car-battery' },
  'Mechanical': { abbreviation: 'Mech', koreanName: '기계', icon: 'fa-solid fa-gears' }  // ← 추가
};
```

> Font Awesome 아이콘은 https://fontawesome.com/icons 에서 검색할 수 있습니다.

#### Step 4: (선택) index.html — `categoryMap`에 카테고리 추가 (374행 부근)

카테고리별 분류가 필요한 경우에만 추가합니다. 필수는 아닙니다.

#### Step 5: Confluence에 배포

1. 새 JSON 파일을 Confluence 페이지에 첨부파일로 업로드
2. 수정된 index.html을 HTML 매크로에 반영

---

## 9. 자주 발생하는 문제와 해결 방법

### 9.1 용어가 표시되지 않는 경우

| 증상 | 원인 | 해결 |
|------|------|------|
| 특정 공종만 안 보임 | 해당 JSON 파일 형식 오류 | JSON 유효성 검증 후 재업로드 |
| 전체 데이터 안 보임 | Confluence 첨부파일 누락 또는 PAGE_ID 불일치 | 첨부파일 목록 확인, PAGE_ID 확인 |
| 새로 추가한 용어 안 보임 | 브라우저 캐시 | Ctrl+Shift+R (강력 새로고침) |
| 새 공종이 안 보임 | jsonFiles 또는 disciplineMap 미등록 | index.html의 두 매핑 객체 확인 |

### 9.2 레이아웃이 깨지는 경우

| 증상 | 원인 | 해결 |
|------|------|------|
| 스타일이 이상함 | Confluence 테마/CSS 충돌 | index.html 하단의 Confluence 호환 CSS 확인 |
| 아이콘이 안 보임 | Font Awesome CDN 접근 불가 | 사내 네트워크에서 CDN 접근 허용 여부 확인 |
| 모바일에서 깨짐 | CSS 미디어 쿼리 문제 | 768px 기준 반응형 CSS 확인 |

### 9.3 브라우저 콘솔에서 오류 확인하는 방법

1. 페이지에서 `F12` 키를 눌러 개발자 도구 열기
2. `Console` 탭 선택
3. 빨간색 오류(error) 또는 노란색 경고(warning) 메시지 확인
4. `Failed to load` 메시지가 있으면 해당 JSON 파일에 문제가 있는 것

---

## 10. 주의사항

### 반드시 기억해야 할 사항

1. **JSON 파일 수정 후에는 반드시 유효성 검증을 해주세요.** 쉼표 하나 빠져도 해당 공종 전체 데이터가 로드되지 않습니다.

2. **파일명을 변경하지 마세요.** `jsonFiles` 매핑과 Confluence 첨부파일명이 정확히 일치해야 합니다. 파일명을 변경하면 index.html의 매핑도 함께 수정해야 합니다.

3. **PAGE_ID를 잘 관리하세요.** Confluence 페이지를 새로 만들거나 이동하면 PAGE_ID가 바뀝니다. index.html 293행의 `PAGE_ID` 값을 새 페이지 ID로 갱신해야 합니다.

4. **Confluence 페이지 ID 확인 방법:**
   - 해당 Confluence 페이지에서 `⋯` → `페이지 정보` (Page Information) 클릭
   - URL에서 `pageId=` 뒤의 숫자 확인
   - 또는 페이지 URL에 `/pages/viewinfo.action?pageId=` 형태로 표시됨

5. **Civil 공종은 4개 파일로 분할되어 있습니다.** 용어를 추가할 때 적절한 파일에 추가하거나, 한 파일이 너무 커지지 않도록 관리해주세요.

6. **Git 저장소도 함께 관리하세요.** 이 프로젝트는 GitHub 저장소(`Nabiyahs/tech-glossary`)에서도 관리됩니다. Confluence 업로드와 별도로, 변경사항을 Git에도 커밋해두면 버전 이력 관리에 도움이 됩니다.

### 외부 의존성

| 항목 | URL | 용도 |
|------|-----|------|
| Font Awesome 6.5.1 | `https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css` | 공종 아이콘 표시 |
| Confluence REST API | `/rest/api/content/{PAGE_ID}/child/attachment` | 첨부파일 조회 |

---

## 부록: 빠른 참고 체크리스트

### 용어만 수정할 때 (가장 빈번)

- [ ] 해당 공종 JSON 파일을 텍스트 에디터로 수정
- [ ] JSON 유효성 검증 (jsonlint.com 등)
- [ ] Confluence 페이지 → 첨부파일 → 동일 이름으로 파일 업로드
- [ ] 페이지 새로고침하여 확인

### index.html을 수정할 때 (드문 경우)

- [ ] index.html 수정
- [ ] Confluence 페이지 편집 → HTML 매크로 본문 교체
- [ ] 페이지 저장 후 확인

### 새 공종을 추가할 때

- [ ] JSON 파일 생성 (규격에 맞게)
- [ ] index.html `jsonFiles` 매핑에 추가
- [ ] index.html `disciplineMap`에 추가
- [ ] Confluence에 JSON 파일 첨부파일 업로드
- [ ] Confluence HTML 매크로에 index.html 반영
- [ ] 페이지 새로고침하여 확인
