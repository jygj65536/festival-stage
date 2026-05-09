# 토스 앱인토스(Apps in Toss) 개발 가이드

> 출처: https://developers-apps-in-toss.toss.im/ | 최종 갱신: 2026-05

---

## 1. 플랫폼 개요

**앱인토스**는 파트너사가 개발한 서비스를 토스 앱 내부에서 '앱 안의 앱(App-in-App)' 형태로 노출할 수 있는 미니앱 플랫폼이다.

- 누적 가입자 **3,000만 명** 규모의 토스 사용자 기반 활용
- 별도 앱 스토어 배포 없이 토스 앱 내에서 직접 서비스 제공
- CDN 호스팅: 빌드 결과물을 콘솔에 업로드하면 토스 CDN 서버가 호스팅

### 최소 지원 환경

| 항목 | 최솟값 |
|------|--------|
| Android | Android 7 이상 |
| iOS | iOS 16 이상 |
| 사용자 연령 | 19세 이상 |

---

## 2. 개발 방식 선택

### 2-1. WebView SDK (권장 - 웹 기반)

- **패키지**: `@apps-in-toss/web-framework`
- 기존 React/Vue/jQuery 등 웹 기술 그대로 사용 가능
- 빠른 개발, 낮은 진입 장벽
- 비게임 서비스에 적합

### 2-2. React Native SDK (네이티브 성능)

- **패키지**: `@apps-in-toss/framework`
- 네이티브 수준의 UX가 필요한 경우
- TDS 버전에 따른 의존성 분기:
  - `@apps-in-toss/framework` < 1.0.0 → `@toss-design-system/react-native`
  - `@apps-in-toss/framework` ≥ 1.0.0 → `@toss/tds-react-native`

### 2-3. 게임 프레임워크

- **Unity**: 플러그인 기반
- **Cocos**: 예시 기반 (`github.com/toss/apps-in-toss-cocos-examples`)

---

## 3. 프로젝트 생성 & 설정

### WebView 신규 프로젝트

```bash
npx create-ait-app <app-name>
```

설정 옵션: TDS 통합, AI 코딩 도구 지원, 예제 코드 포함 여부 선택

### 기존 웹 프로젝트에 추가

```bash
npm install @apps-in-toss/web-framework
npx ait init
```

### React Native 신규 프로젝트

```bash
npm create granite-app
npm install @apps-in-toss/framework
npx ait init
```

### `granite.config.ts` 핵심 설정

```ts
export default {
  appName: "my-app",          // 콘솔 등록명과 동일해야 함 (변경 불가)
  displayName: "내 앱",        // 네비게이션 바에 표시되는 이름
  primaryColor: "#3182F6",     // TDS 컴포넌트 테마 색 (RGB HEX)
  icon: "./assets/icon.png",   // 앱 아이콘 경로
  web: {
    host: "localhost",         // 실기기 테스트 시 네트워크 IP로 변경
    port: 3000,
  },
  permissions: [],             // 필요한 권한 목록
};
```

> **주의**: `appName`은 딥링크(`intoss://{appName}`) 및 배포에 사용되며 **등록 후 변경 불가**

---

## 4. Granite 프레임워크 아키텍처

### 핵심 구성요소

- **Granite**: WebView/RN 양쪽 공통 런타임. 앱 실행 환경 초기화 & 토스 앱과의 통신 담당
- **AppsInToss**: 서비스 환경 등록 인터페이스
  - `registerApp()` — 서비스 환경 초기화, 핵심 기능 활성화
  - `appName` property — 앱 식별자

### InitialProps (네이티브 → 앱 전달 초기 데이터)

| 필드 | 설명 |
|------|------|
| platform | iOS / Android 구분 |
| initialColorPreference | 사용자 색상 테마 |
| networkStatus | 현재 연결 상태 및 네트워크 타입 |
| urlScheme | 현재 화면 진입에 사용된 URL |
| fontConfiguration | 폰트 사이즈 및 접근성 스케일 |
| visibilityState | (iOS only) 화면 현재 표시 여부 |

### 파일 기반 라우팅 (Next.js 스타일)

```
pages/index.tsx       → intoss://app-name
pages/detail.tsx      → intoss://app-name/detail
pages/item/index.tsx  → intoss://app-name/item
```

---

## 5. SDK 주요 기능

### 필수 기능

| 기능 | 설명 |
|------|------|
| 로그인 (앱 로그인) | 유저 식별자(hash) 발급, 회원가입 불필요 |
| Safe Area | 노치/다이나믹 아일랜드 등 기기별 UI 안전 영역 보장 (검수 필수) |
| Native Storage | 앱 종료 후에도 유지되는 영구 저장소 (localStorage보다 안정적) |
| 유저 행동 트래킹 | 참여 패턴 모니터링 분석 함수 |

### 수익화 기능

| 기능 | 설명 |
|------|------|
| 인앱광고 (IAA) | 전면(interstitial), 보상형(rewarded), 배너(banner) 3가지 형식 |
| 인앱결제 (IAP) | 최소 400원 ~ 최대 140만원, 토스 수수료 5% + 앱스토어 수수료 별도 |
| 프로모션 (토스포인트) | SDK 함수 호출만으로 사용자에게 토스 포인트 직접 지급 (서버 불필요) |
| 공유 리워드 | 사용자 간 공유 시 인센티브 제공 |

### 게임 특화 기능

- 리더보드 (랭킹 시스템)
- 햅틱 피드백
- 화면 방향 잠금
- 항상 켜짐 모드
- 오디오 포커스 변경 콜백
- 서버 시간 동기화

### 기타 기능 (비게임)

- 카메라, 연락처, 앨범 사진, 클립보드
- 위치 추적 (콜백, 1회, 연속)
- 네트워크 상태, 로케일 설정
- 공유 (텍스트, 링크)
- 플랫폼 OS 감지

---

## 6. 개발 & 테스트 워크플로우

```
로컬 개발 (npm run dev)
    ↓
샌드박스 앱 테스트 (시뮬레이터/실기기)
    ↓
빌드 (npm run build → <serviceName>.ait 파일 생성)
    ↓
콘솔 업로드 + QR코드 생성
    ↓
토스 앱에서 실기기 최종 테스트
    ↓
검수 신청
```

### 번들 업로드 (CI/CD)

```bash
npx ait deploy --api-key {API_KEY}   # SDK v1.4.0+ 필요
```

### 테스트 전제 조건

- 토스 앱 로그인 상태
- 워크스페이스 멤버 등록
- 19세 이상

### 번들 제한

- **압축 해제 후 100MB 이하**

### 자주 발생하는 이슈

- iOS 흰 화면: Safe Area 미적용
- 통신 실패: CORS 설정 누락
- mTLS 인증서: 서버 간 API 통신 필수

---

## 7. 입점(온보딩) 프로세스 5단계

| 단계 | 내용 | 소요 시간 |
|------|------|-----------|
| 1. 시작 | 콘솔에 앱 등록 | - |
| 2. 디자인 | AppBuilder 또는 Figma로 UI 제작 | - |
| 3. 개발 | WebView/RN/Unity/Cocos로 개발, SDK 연동 | 비게임 1~3개월, 게임 2~4주 |
| 4. 검수 | 운영/기능/디자인/보안 4단계 검토 | 2~3 영업일 |
| 5. 출시 | 콘솔에서 출시 버튼 클릭 | - |

### 검수 4단계

1. **운영 검증**: 정책 준수 여부
2. **기능 테스트**: SDK 통합, 핵심 기능 동작
3. **디자인 컴플라이언스**: TDS 가이드라인 준수
4. **보안 검사**: 취약점 및 데이터 처리

### 앱 등록 검토: 1~2 영업일 / 출시 검토: 3 영업일

---

## 8. 디자인 요구사항

- **TDS(Toss Design System)** 사용 필수 (비게임 WebView/RN 모두)
- **라이트 모드만** 지원 (다크 모드 미지원)
- **금지된 다크 패턴**:
  - 예상치 못한 바텀 시트
  - 불명확한 종료 옵션
  - 불시 광고

---

## 9. 비즈니스 요구사항

- 사업자 등록 없이도 개발 가능하지만, **로그인/토스페이/프로모션/광고** 기능 사용 시 사업자 등록 필수
- 현재 얼리 파트너에게 **서비스 수수료 0%** 적용 중 (향후 광고수익/IAP/결제 처리에 수수료 적용 예정)
- 정산: 사업체 단위로 처리 (앱 단위 아님)
- 세금계산서: 2~3 영업일 내 발행

### 금지 서비스

- 디지털 자산, 자금세탁 위험 서비스
- 불법 활동, 도박 콘텐츠
- 금융상품 중개, 투자 자문
- 데이팅 플랫폼

---

## 10. AI 개발 도구 (AX)

**AX (AppsInToss eXperience)**: MCP 서버 기반 AI 개발 보조 툴킷

```bash
# 설치
brew install toss/tap/ax          # macOS
scoop install ax                  # Windows
npm install -g @apps-in-toss/ax   # npm

# 실행
ax mcp start
```

### Claude / Cursor 연동

`claude_desktop_config.json` 또는 `.cursor/mcp.json`에 AX MCP 서버 설정 추가

**제공 기능**:
- 앱인토스 개발 문서 검색
- TDS React Native / TDS Web 문서 검색
- 코드 예제 조회 및 참조

---

## 11. 핵심 링크

| 자료 | URL |
|------|-----|
| 개발자 센터 | https://developers-apps-in-toss.toss.im/ |
| 콘솔 | https://apps-in-toss.toss.im/ |
| 개발자 커뮤니티 | https://techchat-apps-in-toss.toss.im/ |
| GitHub 예시 (Web) | https://github.com/toss/apps-in-toss-examples |
| GitHub 예시 (Cocos) | https://github.com/toss/apps-in-toss-cocos-examples |
| AX (AI 개발 도구) | https://github.com/toss/apps-in-toss-ax |
| 입점 소개 페이지 | https://toss.im/apps-in-toss |

---

## 12. SDK 마이그레이션 공지

- **SDK 2.x** 지원: React Native 0.84 + React 19
- **SDK 1.x 업로드 마감**: 2026년 3월 23일 → 이미 만료됨
- **신규 프로젝트는 반드시 SDK 2.x 사용**
