# gen-codeflow: Interactive Code Pipeline Flowchart Generator

프로젝트 코드베이스를 분석하여 **인터랙티브 파이프라인 플로우차트 HTML**을 생성하는 방법 기록.
실제 생성 결과: `baseline/2026_GraspGen/pipeline_flowchart.html`

---

## 1. 출력물 스펙

| 항목 | 설명 |
|------|------|
| 형식 | 단일 HTML 파일 (빌드 불필요, 브라우저에서 바로 열기) |
| 프레임워크 | React 18 + Babel standalone (CDN) |
| 외부 의존성 | 없음 — 3개 CDN 스크립트만 사용 |
| UI 구성 | 탭 네비게이션 + 클릭 가능 노드 + 코드 사이드 패널 |
| 뷰어 | 브라우저 (`open file.html`) 또는 VS Code Live Preview 확장 |

## 2. 아키텍처

```
┌──────────────────────────────────────────────────────┐
│ Single HTML File                                      │
│                                                       │
│ <head>                                                │
│   CDN: react.production.min.js                       │
│   CDN: react-dom.production.min.js                   │
│   CDN: @babel/standalone (JSX → JS 변환)             │
│   <style> — 전역 CSS (다크 테마, 코드 하이라이팅)      │
│ </head>                                               │
│                                                       │
│ <body>                                                │
│   <div id="root"/>                                    │
│   <script type="text/babel">                         │
│     1. codeDB         — 코드 스니펫 데이터베이스       │
│     2. nodeCodeMap    — 노드 이름 → codeDB 키 매핑    │
│     3. depNodes/Edges — 의존성 그래프 데이터           │
│     4. Styles (S)     — inline style 객체             │
│     5. Components     — 탭별 React 컴포넌트           │
│     6. App            — 메인 앱 (탭 + 코드 패널)      │
│   </script>                                           │
│ </body>                                               │
└──────────────────────────────────────────────────────┘
```

## 3. 데이터 구조 (3개 핵심)

### 3.1 codeDB — 코드 스니펫 DB

프로젝트의 핵심 코드 조각을 HTML 구문 강조와 함께 저장.

```javascript
const codeDB = {
  "snippet_key": {
    title: "함수/클래스명 — file.py:line_range",
    file: "relative/path/to/source.py",
    code: `<span class="kw">def</span> <span class="fn">function_name</span>(self, arg):
    <span class="cm"># 설명 주석</span>
    result = self.<span class="fn">method</span>(arg)  <span class="cm"># → [B, 512]</span>
    <span class="kw">return</span> result`
  },
  // ... 20~30개 스니펫
};
```

**구문 강조 CSS 클래스:**
| 클래스 | 용도 | 색상 |
|--------|------|------|
| `.kw` | 키워드 (def, class, for, if, return) | `#ff7b72` (red) |
| `.fn` | 함수/메서드명 | `#d2a8ff` (purple) |
| `.cls` | 클래스명 | `#ffa657` (orange) |
| `.str` | 문자열 | `#a5d6ff` (blue) |
| `.num` | 숫자/불리언 | `#79c0ff` (cyan) |
| `.cm` | 주석 | `#8b949e` (gray) |

### 3.2 nodeCodeMap — 노드-코드 매핑

플로우차트의 각 노드 이름을 codeDB 키에 연결.

```javascript
const nodeCodeMap = {
  "Input: Point Cloud": "input_pc",
  "PointNet++ Encode": "pointnet_encode",
  "DDPM Reverse Denoise": "ddpm_denoise",
  // 모든 클릭 가능 노드
};
```

### 3.3 depNodes / depEdges — 의존성 그래프

SVG 기반 graphviz-style 의존성 다이어그램.

```javascript
const depNodes = [
  // id, label, x, y, w, h, color, group
  {id:"module_name", label:"file.py\n(ClassName)", x:400, y:30, w:170, h:44, color:"#1e40af", group:"models"},
];

const depEdges = [
  // [from_id, to_id, edge_label]
  ["parent_module", "child_module", "uses"],
];

const groupColors = { models:"#1e3a5f", utils:"#134e4a", /* ... */ };
const groupLabels = { models:"src/models/", utils:"src/utils/", /* ... */ };
```

## 4. UI 컴포넌트 패턴

### 4.1 탭 구조

프로젝트 특성에 맞게 3~6개 탭으로 구성.

```
일반적인 탭 구성:
├── Inference Pipeline  — 입력 → 처리 → 출력 흐름
├── Model Internals     — 학습 과정, loss, 내부 구조
├── Module Registration — 플러그인/확장 등록 패턴
├── Training Pipeline   — 데이터 준비 → 학습 → 평가
└── Dependency Graph    — SVG 모듈 의존성 그래프
```

### 4.2 Node 컴포넌트

클릭하면 코드 패널이 열리는 인터랙티브 노드.

```jsx
const Node = ({title, color, badge, badgeBg, children, onClick}) => {
  const codeKey = nodeCodeMap[title];  // 매핑 존재 시 클릭 가능
  const clickable = !!codeKey;
  return (
    <div style={nodeStyle(color, clickable)}
         onClick={() => clickable && onClick(codeKey)}>
      {clickable && <div className="click-hint">click</div>}
      <div className="title">{badge && <Badge>{badge}</Badge>}{title}</div>
      <div className="body">{children}</div>
    </div>
  );
};
```

### 4.3 CodePanel 컴포넌트

우측 42% 영역에 코드 스니펫을 표시하는 사이드 패널.

```jsx
const CodePanel = ({codeKey, onClose}) => {
  const entry = codeDB[codeKey];
  return (
    <div style={{width:"42%", position:"sticky", top:0, height:"100vh"}}>
      <div className="header">
        <span>{entry.title}</span>
        <button onClick={onClose}>✕</button>
      </div>
      <div className="file-path">{entry.file}</div>
      <pre dangerouslySetInnerHTML={{__html: entry.code}}/>
    </div>
  );
};
```

### 4.4 Dependency Graph (SVG)

```jsx
const DepGraph = ({onCodeClick}) => {
  const [hover, setHover] = useState(null);
  return (
    <svg width={880} height={580}>
      {/* Group 배경 rect */}
      {/* Edge 선 (hover 시 하이라이트) */}
      {/* Node rect + text (클릭 시 코드 패널 연결) */}
    </svg>
  );
};
```

## 5. 생성 프로세스 (Step-by-step)

### Step 1: 코드베이스 탐색

```
목표: 프로젝트의 모듈 구조와 데이터 흐름을 파악

탐색 대상:
├── 진입점 (main, demo scripts, CLI)
├── 핵심 모델/클래스 (forward/inference 메서드)
├── 데이터 파이프라인 (dataset, dataloader, preprocessing)
├── 설정 시스템 (config, registry, plugin)
├── 유틸리티 (visualization, utils)
└── 외부 의존성 (CUDA ops, C++ extensions)

방법:
- Explore agent로 3개 병렬 탐색 (models, data, config)
- 각 모듈에서 class, method signature, tensor shape 추출
- 모듈 간 import 관계 → depEdges 생성
```

### Step 2: 파이프라인 흐름 정의

```
입력 → 전처리 → 모델 추론 → 후처리 → 출력

각 단계에서:
- 입력/출력 텐서 shape
- 핵심 함수/클래스 + 파일:라인
- 분기점 (옵션, 조건부 실행)
- 설정 파라미터
```

### Step 3: 코드 스니펫 작성

```
원칙:
- 원본 코드를 직접 복사하되 핵심 로직만 추출 (20~40줄)
- 텐서 shape를 주석으로 표기 (# → [B, 512])
- HTML span 태그로 구문 강조 (수작업)
- 설정 값/하이퍼파라미터 포함
```

### Step 4: 탭 구성 + 노드 배치

```
탭 수: 프로젝트 복잡도에 따라 3~6개
노드 수: 탭당 3~8개
코드 스니펫: 전체 20~35개

배치 원칙:
- 데이터 흐름 순서: 위 → 아래
- 병렬 구조: grid로 횡배치 (cols(2) or cols(3))
- 그룹 구분: 색상 + 배지 (INPUT, MODEL, LOSS, OUTPUT)
```

### Step 5: 의존성 그래프 레이아웃

```
수동 좌표 배치 (자동 레이아웃 아님):
- 노드: {x, y, w, h} — 픽셀 단위
- 그룹: 배경 rect로 디렉토리 구조 시각화
- 엣지: 직선 (line), 화살표 마커 (SVG marker)
- 상호작용: hover → 엣지 하이라이트, click → 코드 패널

레이아웃 팁:
- 상위 모듈을 위에, 하위를 아래에 (계층 구조)
- 같은 디렉토리 = 같은 그룹 배경
- 엣지가 교차하지 않도록 좌표 조정
- 캔버스: 880×580 정도 (스크롤 없이 한 화면)
```

## 6. 스타일 가이드

### 다크 테마 팔레트

```css
/* 배경 계층 */
body:     #0f172a  (최하위)
card:     #1e293b  (카드)
panel:    #161b22  (코드 패널)
code-bg:  #0d1117  (코드 블록)

/* 텍스트 */
primary:  #f8fafc
secondary:#94a3b8
dim:      #64748b

/* 강조 */
blue:     #3b82f6 (클릭 가능, 선택)
green:    #22c55e (출력, 성공)
orange:   #f59e0b (처리 단계)
red:      #dc2626 (학습, 에러)
purple:   #8b5cf6 (단계 번호)
```

### 반응형 레이아웃

```javascript
// 코드 패널 열림 시 메인 영역 축소
main:  { flex:1, maxWidth: panelOpen ? "58%" : "100%" }
panel: { width: "42%", position:"sticky", top:0, height:"100vh" }
```

## 7. CDN 의존성

```html
<script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
```

- React 18: UI 컴포넌트
- ReactDOM 18: DOM 렌더링
- Babel standalone: `<script type="text/babel">` → 브라우저에서 JSX 변환
- **오프라인 사용 불가** (CDN 필요)

## 8. 실행 방법

```bash
# 브라우저에서 직접 열기
open pipeline_flowchart.html           # macOS
xdg-open pipeline_flowchart.html       # Linux

# VS Code Live Preview 확장
# 1. Extensions → "Live Preview" 설치
# 2. HTML 파일 우클릭 → "Show Preview"

# Python HTTP 서버 (옵션)
python -m http.server 8000
# → http://localhost:8000/pipeline_flowchart.html
```

## 9. 체크리스트 (생성 시)

```
[ ] Step 1: 코드베이스 탐색 (Explore agent × 3 병렬)
    [ ] 모듈 구조 파악
    [ ] 핵심 클래스/함수 목록
    [ ] 데이터 흐름 (입력 → 출력)
    [ ] 모듈 간 import 의존성
[ ] Step 2: 파이프라인 흐름 정의
    [ ] 탭 수/구성 결정
    [ ] 각 탭의 노드 목록
    [ ] 노드 간 화살표 (데이터 흐름)
[ ] Step 3: codeDB 작성 (20~35개 스니펫)
    [ ] 각 스니펫: title, file, code (HTML 구문 강조)
    [ ] 텐서 shape 주석
[ ] Step 4: nodeCodeMap 작성
    [ ] 모든 클릭 가능 노드 → codeDB 키 매핑
    [ ] 중복 없는지 확인
[ ] Step 5: 의존성 그래프
    [ ] depNodes (id, label, x, y, w, h, color, group)
    [ ] depEdges (from, to, label)
    [ ] groupColors / groupLabels
    [ ] 노드 클릭 → codeDB 매핑
[ ] Step 6: HTML 생성 + 검증
    [ ] 브라우저에서 열기
    [ ] 모든 탭 전환 확인
    [ ] 모든 노드 클릭 → 코드 패널 확인
    [ ] 의존성 그래프 hover/click 확인
```

## 10. 참고: GraspGen 생성 통계

| 항목 | 수량 |
|------|------|
| 탭 수 | 5 |
| codeDB 스니펫 | 22 |
| 클릭 가능 노드 | 22 |
| depNodes | 19 |
| depEdges | 21 |
| 그룹 (디렉토리) | 7 |
| HTML 파일 크기 | ~930줄, ~45KB |
| 생성 시간 | 코드 분석 2시간 + HTML 생성 30분 |
