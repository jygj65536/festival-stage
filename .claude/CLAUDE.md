# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **festival information mini-app** being built for the Toss "Apps in Toss" (앱인토스) platform — a mini-app platform embedded inside the Toss financial app (30M+ users). The project is in the planning phase; no application code exists yet.

- **Team**: 2 people — A (planning + design), B (development, beginner, AI-assisted)
- **Target platform**: Apps in Toss (WebView SDK route)
- **Goal**: List festival info, timetables, venue maps, and push notifications for upcoming performances

## Planned Tech Stack

| Layer | Choice |
|-------|--------|
| Framework | React + TypeScript via `create-ait-app` |
| Apps in Toss SDK | `@apps-in-toss/web-framework` (WebView, SDK 2.x) |
| Styling | Tailwind CSS |
| Backend / DB | Supabase (tables: `festivals`, `stages`, `performances`) |
| Deployment | Vercel (auto-deploy on GitHub push) |
| Push scheduler | Supabase Edge Functions |

## Bootstrap Commands (once development starts)

```bash
npx create-ait-app <app-name>   # scaffold new project
npm run dev                      # local dev server
npm run build                    # produces <serviceName>.ait bundle
npx ait deploy --api-key {KEY}  # upload bundle to Toss console (SDK v1.4.0+)
```

## Apps in Toss Platform Constraints

### Hard rules
- `appName` in `granite.config.ts` **cannot be changed** after console registration — confirm before registering
- SDK 1.x bundles are rejected since 2026-03-23; use **SDK 2.x only**
- Bundle size limit: **100 MB** uncompressed
- Business registration is required for: Toss Login, Toss Pay, promotions, in-app ads

### Automatic review rejections
- Missing or inaccessible privacy policy / terms of service URLs
- API keys hardcoded in source (use env vars)
- HTTP (non-HTTPS) endpoints
- Toss navigation bar customized or hidden
- Buttons that route users to external apps or external payment pages
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
- **Maps**: Plain SVG + React (no mapping library needed; design asset from designer required first)
- **Data entry**: Supabase dashboard allows non-developer data updates without code changes — design the schema to support this

## Development Workflow

```
Local dev → Sandbox app test (simulator + real device)
→ npm run build → console upload → QR test on real Toss app
→ Submit for review (2–3 business days)
```

## AI Tool Integration

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
