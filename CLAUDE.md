# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Commands

- `npm run dev` / `npm run build` / `npm start` — Next.js 16 (Turbopack by default)
- `npm run lint` — ESLint 9 flat config (`eslint.config.mjs`: `core-web-vitals` + `typescript`)
- `npx tsc --noEmit` — typecheck (no script defined)
- `npx shadcn@latest add <component>` — add UI components
- No test runner configured yet.

## Workflow

**`docs/SETUP.md` is the source of truth** for project rules — read it before writing code:
- §1 Folder structure: domain modules under `modules/<domain>/`, English names, kebab-case files, `app/` is routing only.
- §2 Best practices: SOLID/DRY/KISS/YAGNI; check shadcn first; search for an existing component/function/hook before creating one.
- §3 Methodology: SDD (Spec Driven Development) with unit tests where the spec requires them.

### SDD agents (`.claude/agents/`)

| Agent | File | Role |
| ----- | ---- | ---- |
| `orchestrator` | `.claude/agents/orchestrator.md` | Entry point. Classifies each task as BUILD (direct, 1–3 files, no new contracts) or SDD, plans achievable phases, dispatches the others (in parallel when task `Owns` don't overlap), runs the review loop (max 3 rounds). |
| `spec` | `.claude/agents/spec.md` | Writes `docs/specs/<module>/<feature>.md`: ACs, contracts, reuse analysis, phased plan with per-task file ownership. Always `draft`. |
| `developer` | `.claude/agents/developer.md` | Implements one approved task, touching only its `Owns`. |
| `reviewer` | `.claude/agents/reviewer.md` | Read-only. Validates against spec + SETUP; returns `APPROVED` / `CHANGES_REQUESTED` / `SPEC_ISSUE`. |

Run with `claude --agent orchestrator` (recommended, it can ask you for approval directly). It also works as a subagent, handing approvals back to the main thread.

**Human approval is blocking:** no `developer` work starts until a human explicitly approves the spec and it has `Status: approved` + `Approved by: <name> (<date>)`. The developer and reviewer both refuse specs without it. Specs that change criteria/contracts/plan go back to `draft` and need re-approval.

## Stack and conventions

Empty template: Next.js 16 App Router (`app/`), React 19, TypeScript strict, Tailwind 4 (CSS-first, configured in `app/globals.css` via `@tailwindcss/postcss`; there is no `tailwind.config`). Path alias `@/*` maps to the repo root.

Installed but **not wired up yet**: `@tanstack/react-query`, `@tanstack/react-table` (v9), `zustand`, `axios`, `zod` (v4). Nothing provides a `QueryClientProvider` or stores; add them when first needed.

### shadcn/ui

`components.json` uses the `base-nova` preset, which is built on **`@base-ui/react`** (not Radix) — Radix-style props/docs from older shadcn examples may not apply. Aliases: `@/components/ui`, `@/lib/utils`, `@/hooks`. Icons: `lucide-react`.

`cn()` comes from the `cn` npm package (maintained by shadcn, replaces `clsx` + `tailwind-merge`), re-exported by `lib/utils.ts`. Don't add `clsx`/`tailwind-merge`.
