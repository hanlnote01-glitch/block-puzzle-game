# 🧩 블록 퍼즐 (Block Puzzle)

8×8 보드에 3개씩 주어지는 블록 조각을 끌어다 놓고, 가로·세로 한 줄을 채워 지우는
1010! / 블록 블라스트 스타일 퍼즐 게임입니다. 화면 맨 위에는 **상시 노출되는 320×50 배너 광고 영역**이
게임 레이아웃과 분리되어 고정되어 있습니다.

- 순수 HTML/CSS/JS **단일 파일** — 빌드 도구·서버·인터넷 없이 바로 실행됩니다.
- 모바일(터치) · PC(마우스) 동시 지원, 세로 화면 기준(Safe Area 대응).
- 점수·최고 점수는 `localStorage` 에 저장됩니다.

---

## 1. 실행 방법

가장 간단한 방법은 `index.html` 파일을 브라우저로 여는 것입니다.

```bash
# 1) 파일을 그대로 열기
#    (Windows/macOS: index.html 더블클릭, Android: 파일 관리자에서 Chrome으로 열기)

# 2) 또는 로컬 정적 서버로 열기 (manifest/PWA 동작 확인용)
cd projects/block-puzzle-game
python3 -m http.server 8080
# → 브라우저에서 http://localhost:8080 접속
```

> `file://` 로 열어도 게임은 100% 동작합니다. 다만 `manifest.json`(홈 화면 추가)과
> 광고 스크립트는 `http(s)` 환경에서만 정상 동작합니다.

---

## 2. 파일 구조

```
block-puzzle-game/
├── index.html          # 게임 전체 (HTML + CSS + JS 단일 파일)
├── manifest.json       # PWA(홈 화면 추가) 매니페스트
├── icons/
│   ├── icon-192.png
│   └── icon-512.png
├── README.md
└── tests/
    └── verify.py       # 문법·로직·UI 플로우 자동 검증 스크립트
```

`index.html` 안의 스크립트는 세 블록으로 나뉘어 있습니다.

| 블록 | 내용 |
| --- | --- |
| `<script id="ad-config">` | **AD_CONFIG** — 광고 설정 (파일 최상단) |
| `<script id="block-core">` | `BlockCore` — 게임 규칙 순수 로직 (DOM 의존 없음, 테스트 대상) |
| `<script id="block-ui">` | 화면 렌더링 · 드래그 앤 드롭 · 사운드 · 광고 마운트 |

---

## 3. 게임 규칙과 점수

| 항목 | 내용 |
| --- | --- |
| 보드 | 8×8 (64칸) |
| 조각 | 하단 트레이에 항상 3개, 29종(1칸 ~ 3×3) 무작위 |
| 배치 | 조각을 드래그해 빈 칸에 놓기. 놓을 수 없는 위치는 하이라이트가 뜨지 않음 |
| 줄 제거 | 가로 또는 세로 한 줄이 꽉 차면 제거 |
| 리필 | 3개를 모두 쓰면 새 3개 생성 (최소 1개는 반드시 놓을 수 있게 보장) |
| 게임 오버 | 남은 조각 중 어느 것도 놓을 수 없을 때 → **다시하기** 버튼 |

### 점수

| 동작 | 점수 |
| --- | --- |
| 조각 배치 | 조각의 칸 수 × 1점 |
| 1줄 제거 | 100점 |
| 2줄 동시 제거 | 300점 |
| 3줄 동시 제거 | 600점 |
| 4줄 동시 제거 | 1,000점 |
| 5줄 이상 | 100 × n(n+1)/2 |
| 콤보 | 연속으로 줄을 지울 때마다 배수 ×1, ×2, ×3 … |
| 완벽 정리 | 줄 제거 후 보드가 완전히 비면 +500점(× 콤보 배수) |

콤보는 줄을 지우지 못한 배치가 한 번 들어오면 초기화됩니다.

---

## 4. 상단 배너 광고 설정

배너는 `.ad-slot` (높이 56px, 최대 너비 360px, 중앙 정렬) 안에
`.ad-box` (320×50) 로 들어가며, `flex` 세로 레이아웃의 첫 번째 요소라 보드와 절대 겹치지 않습니다.
광고 ID가 없을 때는 아래처럼 테스트 배너가 표시됩니다.

```
AD · 320×50 상단 배너 광고 영역
```

### 4-1. Google AdSense (웹) 연결

`index.html` 맨 위 `<script id="ad-config">` 의 `AD_CONFIG` 만 고치면 됩니다.

```js
var AD_CONFIG = {
  enabled: true,
  provider: "adsense",                 // ★ test → adsense
  width: 320,
  height: 50,
  adsense: {
    client: "ca-pub-0000000000000000", // ★ 본인 게시자 ID
    slot: "1234567890",                // ★ 광고 단위 ID (320×50 디스플레이)
    format: "horizontal",
    test: false
  },
  ...
};
```

1. [AdSense](https://adsense.google.com) 에서 사이트를 승인받습니다(도메인 소유 확인 필요).
2. **광고 → 광고 단위별 → 디스플레이 광고**에서 **가로형 320×50** 단위를 만들고
   `data-ad-client`(ca-pub-…)와 `data-ad-slot`(숫자)을 복사합니다.
3. 위 `client` / `slot` 에 붙여넣고 `provider` 를 `"adsense"` 로 바꿉니다.
4. 사이트 루트에 `ads.txt` 를 추가합니다: `google.com, pub-0000000000000000, DIRECT, f08c47fec0942fa0`
5. 실제 광고가 뜨기 전에는 빈 영역이 남을 수 있습니다. AdSense 정책상 **자동 새로고침(refreshSeconds)은
   0(사용 안 함)** 으로 두는 것을 권장합니다.

> 광고 차단기 사용 시 스크립트가 막혀도 게임 진행에는 전혀 영향이 없습니다(배너 영역만 비어 있음).

### 4-2. AdMob (앱) 연결

**AdMob은 앱 전용이라 웹 페이지에서 직접 호출할 수 없습니다.** 웹 게임을 앱(WebView)으로 감쌀 때
네이티브 쪽에서 배너를 띄우고, JS 브리지로 요청하는 방식으로 연결합니다.

```js
provider: "admob",
admob: { bannerUnitId: "ca-app-pub-0000000000000000/1234567890" }
```

앱(Android WebView / Capacitor) 쪽에서 아래 중 하나를 노출하면 게임이 자동으로 호출합니다.

```java
// Android (Kotlin/Java) — WebView 에 추가
webView.addJavascriptInterface(new Object() {
    @JavascriptInterface public void showBanner(String unitId) { /* AdMob 배너 로드 */ }
}, "AndroidAds");
```

```js
// iOS (WKWebView) — window.webkit.messageHandlers.ads.postMessage({action:'showBanner', unitId})
```

브리지가 없으면 자동으로 테스트 배너로 대체됩니다.

---

## 5. 웹에 배포하기 (무료)

### GitHub Pages

```bash
cd projects/block-puzzle-game
git init
git add .
git commit -m "블록 퍼즐 게임"
git branch -M main
git remote add origin https://github.com/<사용자명>/block-puzzle.git
git push -u origin main
```

GitHub 저장소 → **Settings → Pages → Source: Deploy from a branch → main / (root)** →
`https://<사용자명>.github.io/block-puzzle/` 로 접속됩니다.
(AdSense 승인도 이 도메인으로 신청하면 됩니다.)

### 그 밖의 무료 호스팅

Netlify Drop(폴더를 끌어다 놓기), Cloudflare Pages, Vercel — 모두 `index.html` 이 있는 폴더를 그대로 올리면 됩니다.

---

## 6. 안드로이드 앱(APK)으로 만들기

### 방법 A — Capacitor (권장, AdMob 연동 쉬움)

```bash
npm create @capacitor/app block-puzzle-app
cd block-puzzle-app
# www 폴더에 index.html, manifest.json, icons 를 복사
npm i @capacitor/core @capacitor/cli @capacitor/android
npx cap add android
npx cap sync
# Android Studio 에서 열고 서명 후 Build > Generate Signed Bundle / APK
```

AdMob 배너는 `@capacitor-community/admob` 플러그인의 `AdMob.showBanner()` 를
4-2의 `window.AndroidAds` 브리지에 연결하면 됩니다.

### 방법 B — Bubblewrap (PWA → TWA, 코드 수정 없음)

```bash
npm i -g @bubblewrap/cli
bubblewrap init --manifest https://<사용자명>.github.io/block-puzzle/manifest.json
bubblewrap build          # app-release-signed.apk 생성
```

> TWA 는 웹을 그대로 감싸므로 광고는 AdSense(웹) 방식이 그대로 동작합니다.
> 두 방법 모두 `icons/icon-512.png` 를 앱 아이콘으로 사용합니다.

---

## 7. 직접 검증하기

브라우저 없이도 문법·게임 로직·UI 플로우를 자동 검증할 수 있습니다.

```bash
pip install quickjs          # JS 엔진 (최초 1회)
cd projects/block-puzzle-game
python3 tests/verify.py
```

검증 항목:

1. **JS 문법** — `ad-config` / `block-core` / `block-ui` 세 블록 컴파일 오류 0건
2. **HTML 구조** — 배너 규격(56px·320×50·중앙 정렬), `flex` 레이아웃, 8×8 그리드, 한국어 UI 문구
3. **BlockCore 단위 테스트** — 보드 초기화, 배치 경계·충돌, 보드 불변성, 줄 감지/제거(교차점),
   점수표, 콤보·완벽 정리 계산, 조각 정의 유효성, `hasAnyMove`, 세트 생성 공정성(300회),
   규칙 전체 랜덤 플레이 200판
4. **UI 플로우 테스트** — DOM 셰임 위에서 실제 `index.html` 코드를 실행해
   초기화 · 테스트 배너 문구 · 배치 · 가로/세로 줄 제거 · 콤보 x2 · 완벽 정리 · 리필 ·
   게임 오버 · 다시하기 · 최고 점수 저장 · **드래그(터치 56px 리프트/마우스 12px)** ·
   보드 밖 드롭 무시 · 끝까지 플레이까지 검사

최근 실행 결과: **전체 35항목 통과, JS 오류 0건, 런타임 오류 0건**

---

## 8. 손보기 좋은 지점

| 위치 | 설명 |
| --- | --- |
| `AD_CONFIG` | 광고 provider / ID / 크기 |
| `PALETTE` (block-ui) | 블록 색상 7종 (`c1`, `c2`, `glow`) |
| `SHAPE_DEFS` (block-core) | 조각 29종과 등장 가중치 `weight` |
| `LIFT_TOUCH` (block-ui) | 손가락 위 프리뷰 높이 (기본 56px) |
| `lineScore()` | 줄 점수표 |
| `CLEAR_MS` | 줄 제거 애니메이션 시간(ms) |

---

## 9. 참고

- 사운드는 Web Audio API 로 실시간 합성합니다(별도 음원 파일 없음). 첫 터치 이후부터 재생됩니다.
- 음소거 상태는 `localStorage` 에 저장되어 다음 실행에도 유지됩니다.
- 광고 영역은 게임 오버 오버레이보다 위에 남아 있어 플레이 중에도 항상 노출됩니다.
