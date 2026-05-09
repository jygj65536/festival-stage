# festival-stage

국내 페스티벌 정보 미니앱 — [앱인토스](https://toss.im/apps-in-toss) 입점 프로젝트

## 디렉토리 구조

```
festival-stage/
├── app/          # 프론트엔드 (create-ait-app, React + TypeScript)
├── supabase/
│   ├── functions/    # Edge Functions (Push 알림 스케줄러 등)
│   └── migrations/   # DB 마이그레이션
├── design/       # SVG 지도 에셋, 아이콘 등 디자인 파일
└── docs/
    ├── roadmap.md          # 프로젝트 로드맵 & 실행 계획서
    └── toss-miniapp-guide.md  # 앱인토스 개발 가이드 요약
```

## 시작하기

```bash
# 앱 개발 환경 초기화 (최초 1회)
cd app
npx create-ait-app .

# 로컬 개발 서버
npm run dev

# 빌드 & 콘솔 업로드
npm run build
npx ait deploy --api-key {API_KEY}
```

## 관련 링크

- [앱인토스 개발자 센터](https://developers-apps-in-toss.toss.im/)
- [앱인토스 콘솔](https://apps-in-toss.toss.im/)
- [개발자 커뮤니티](https://techchat-apps-in-toss.toss.im/)
