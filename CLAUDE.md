# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A **festival information mini-app** for the Toss "Apps in Toss" (앱인토스) platform — embedded inside the Toss app (30M+ users). No app store distribution needed.

- **Team**: 2 people — A (planning + design), B (development, beginner, AI-assisted)
- **Target platform**: Apps in Toss (WebView SDK route)
- **Goal**: Festival listings, timetables, venue maps, and push notifications for upcoming performances

## Planned Tech Stack

| Layer | Choice |
|-------|--------|
| Framework | React + TypeScript via `create-ait-app` |
| Apps in Toss SDK | `@apps-in-toss/web-framework` (WebView, SDK 2.x) |
| Styling | Tailwind CSS |
| Backend / DB | Supabase (tables: `festivals`, `stages`, `performances`) |
| Deployment | Vercel (auto-deploy on GitHub push) |
| Push scheduler | Supabase Edge Functions |

## Repository Structure

```
festival-stage/
├── app/              # Frontend — scaffold with create-ait-app, then develop here
├── supabase/
│   ├── functions/    # Edge Functions (push notification scheduler, etc.)
│   └── migrations/   # DB schema migrations
├── design/           # SVG map assets and icons (exported from Figma)
└── docs/
    ├── roadmap.md          # Full project roadmap and execution plan (Korean)
    └── toss-miniapp-guide.md
```

## Commands

```bash
# First-time app setup (run once inside app/)
cd app && npx create-ait-app .

# Local dev server
npm run dev

# Build & upload to Toss console
npm run build
npx ait deploy --api-key {API_KEY}   # requires SDK v1.4.0+
```

## Apps in Toss Platform Constraints

### Hard rules
- `appName` in `granite.config.ts` **cannot be changed** after console registration — confirm before registering
- SDK 1.x bundles rejected since 2026-03-23; use **SDK 2.x only**
- Bundle size limit: **100 MB** uncompressed
- Business registration required for: Toss Login, Toss Pay, promotions, in-app ads

### Automatic review rejections
- Missing or inaccessible privacy policy / terms of service URLs
- API keys hardcoded in source (use env vars)
- HTTP endpoints (HTTPS required everywhere)
- Toss navigation bar modified or hidden
- Buttons routing users to external apps or external payment pages
- Unauthorized third-party analytics/logging SDKs
- UI broken on Galaxy S or iPhone (test both)

### mTLS
Server-to-server API calls to Apps in Toss require an mTLS certificate issued from the console. Without it, all platform APIs fail.

### Safe Area
iOS white screen on launch is almost always a missing Safe Area implementation — apply it to every page root.

## Key Architecture Points

- **Routing**: File-based (Next.js style). `pages/index.tsx` → `intoss://<appName>`, `pages/detail.tsx` → `intoss://<appName>/detail`
- **App init**: Call `registerApp()` from `AppsInToss` on startup to initialize the Granite runtime
- **Storage**: Use Native Storage (SDK) instead of `localStorage` for persistence across app restarts
- **Login**: Toss Login returns a hashed user key; store it in Supabase. Core features must work without login
- **Maps**: Plain SVG + React (no mapping library); design asset from designer required before development starts
- **Data entry**: Design the Supabase schema so non-developers can update festival data directly from the Supabase dashboard without touching code

## Development Workflow

```
Local dev → Sandbox app test (simulator + real device)
→ npm run build → console upload → QR test on real Toss app
→ Submit for review (2–3 business days)
```

## AI Tool Integration (AX MCP)

The Apps in Toss AX MCP server provides documentation search and code examples directly in Claude/Cursor:

```bash
brew install toss/tap/ax   # macOS
ax mcp start
```

Add the AX MCP server to `.cursor/mcp.json` or Claude Desktop config to enable it.

## Key References

- Developer docs: https://developers-apps-in-toss.toss.im/
- Console: https://apps-in-toss.toss.im/
- Developer community (fast QA): https://techchat-apps-in-toss.toss.im/
- Code examples: https://github.com/toss/apps-in-toss-examples
