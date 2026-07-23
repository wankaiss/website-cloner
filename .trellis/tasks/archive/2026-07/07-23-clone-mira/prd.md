# Clone Mira foreign-trade AI assistant (all 8 views)

## Goal

Reverse-engineer and rebuild the **Mira** app at `https://ghiacg.com/` — a B2B
foreign-trade AI assistant SaaS (外贸智能助手) — as a pixel-perfect Next.js clone
with mock data. Deliver all 8 navigation views plus the shared app shell and the
login screen.

User value: a faithful, runnable replica of the Mira product UI for
demonstration and as a design reference, built on the project's
Next.js 16 + React 19 + shadcn/ui + Tailwind v4 scaffold.

## Background

- Target: `https://ghiacg.com/frontend/index.html` (SPA, gates behind login at
  `/frontend/login.html`). App brand: "Mira AI / 外贸智能助手".
- Login required; credentials provided (see memory `ghiacg-login-credentials`).
  Login verified working via Playwright MCP (headless chromium).
- Layout shell: dark-navy fixed sidebar (220px) + white content area; blue→cyan
  brand gradient; modern, dense dashboard aesthetic.
- **The original app itself uses Tailwind utility classes** (confirmed in DOM),
  so extraction can capture near-exact class strings, not just computed CSS.

## Confirmed facts

- Tech stack (target, observed): SPA, Tailwind CSS, SVG icons (Lucide-like),
  no `<canvas>` on dashboard — the "3D globe" (公共库地域分布, "drag to rotate")
  and the email bar chart need library/tech identification during extraction
  (candidate: three.js / globe.gl / echarts / pure SVG).
- Tech stack (clone): Next.js 16 (App Router), React 19, TypeScript strict,
  shadcn/ui, Tailwind v4 (oklch tokens), Lucide. Project scaffold already in
  place and builds.
- Project conventions: see `AGENTS.md` (named exports, PascalCase, camelCase
  utils, 2-space indent, mobile-first, no inline styles, no `any`). Frontend
  specs under `.trellis/spec/frontend/` are empty templates — `AGENTS.md` is the
  binding source.
- Browser automation: Playwright MCP via `@playwright/mcp` pointed at
  `/usr/bin/chromium`, headless. See memory `playwright-mcp-chromium-on-kali`.
- All 8 views are client-side view swaps on the same `index.html` (no per-view
  URL). The clone will introduce real Next.js routes per view.

## View inventory (scope)

| # | View (zh) | Slug | Content | Complexity |
|---|-----------|------|---------|------------|
| 1 | 企业仪表盘 | dashboard | KPI cards, 3D globe, Top-10 list, hot tags, email bar chart, config checklist, quick-link cards | High |
| 2 | 搜索获客 | search-acquisition | Search-task builder + daily task cards | Med |
| 3 | 公共客户池 | customer-pool | Filterable customer grid (country multi-select, actions) | High |
| 4 | 喜好管理 | scoring | Customer scoring rules (many inputs) | Med (form) |
| 5 | 客户管理 | customer-mgmt | Customer list + tags + 背景调查 detail | High |
| 6 | 营销中心 | marketing | SMTP config, auto-marketing tasks, email templates | Med |
| 7 | 对话助手 | chat-assistant | Conversation list + intent filters + auto-reply | Med |
| 8 | 贸易搜索 | trade-search | Import/export buyer search form | Low |

Plus: **login screen** (`login.html`) and the **shared shell** (sidebar + header).

## Requirements

- R1 — Pixel-perfect emulation of every view at desktop (1440) and mobile (390);
  tablet (768) where layouts shift. Exact colors, spacing, typography, radii,
  shadows, states (hover/active/disabled/empty/loading).
- R2 — Shared app shell replicated once and reused: collapsible sidebar, header
  banner, credits widget, routing between the 8 views (Next.js App Router).
- R3 — All interactive behaviors preserved: nav switching, sidebar collapse,
  button/tab/dropdown/modal interactions, hover states, the draggable globe,
  chart hover tooltips, scroll behaviors. Interaction model per section
  documented and matched (click vs scroll vs time).
- R4 — Real assets: download fonts, icons (extract inline SVGs as React
  components), images, favicons. Real UI text/labels verbatim.
- R5 — Mock data layer: realistic data filling each view (see Open Questions for
  the realism/anonymization decision).
- R6 — Foundation first: global CSS tokens, types, icons, fonts, downloaded
  assets — built and `npm run build` green before any view component dispatch.
- R7 — Extraction produces auditable spec files in
  `docs/research/ghiacg/components/<name>.spec.md` (computed CSS via
  `getComputedStyle` + captured Tailwind classes + states + verbatim text);
  builder agents build from these specs in isolated worktrees, merged by the
  orchestrator. Every builder verifies `npx tsc --noEmit`; orchestrator verifies
  `npm run build` after each merge.
- R8 — English/Chinese: the app has an EN/中 toggle. Default to Chinese (current
  live state); i18n toggle scope decided in Open Questions.

## Acceptance criteria

- [ ] `npm run check` (lint + typecheck + build) is green.
- [ ] All 8 views + login + shell render pixel-close to the live site at 1440 and
  390 (visual QA diff per view, discrepancies logged).
- [ ] Sidebar nav switches between all 8 routes; sidebar collapse works.
- [ ] Globe renders and is draggable; bar chart renders with correct series.
- [ ] No real backend/auth/network — fully static mock data; runs on `npm run dev`.
- [ ] Every view has a corresponding spec file under
  `docs/research/ghiacg/components/`.
- [ ] Assets (fonts, icons, images, favicons) are local under `public/`.

## Out of scope

- Real backend, database, authentication, real-time/websocket features.
- Actual AI/search/email functionality (all mocked).
- SEO optimization, accessibility audit (per clone-website skill defaults).
- Reproducing server-generated or per-session-unique content beyond a fixed mock.

## Open questions

1. **Mock-data realism vs anonymization.** The live site shows the account
   holder's real data: their own company (Qingdao Smiler Import and Export
   Co.Ltd), real customer leads (third-party company names, countries), a real
   SMTP mailbox (lisan@ghiacg.com), and real email-marketing stats. Should the
   clone reproduce this data verbatim, or anonymize third-party PII while keeping
   structure/labels/the user's own company intact? (Asked first — affects every
   view.)

## Planned task structure

Parent task `07-23-clone-mira` owns the requirement set, integration, and final
QA. Child tasks (created during planning): `foundation` (shell + tokens + assets
+ types + icons), then one child per view (dashboard, search-acquisition,
customer-pool, scoring, customer-mgmt, marketing, chat-assistant, trade-search),
plus `login`. Foundation is a dependency for all view children.
