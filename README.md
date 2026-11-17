# Lime Payroll Enterprise

[![Live Interactive Sandbox](https://img.shields.io/badge/Live%20Demo-enterprise--payroll.vercel.app-0f766e?style=flat-square)](https://enterprise-payroll.vercel.app/)

A multi-tenant payroll and POS reconciliation web application designed for shift-based hospitality operations—built to replace manual spreadsheet cross-referencing across complex contract rules, tip pooling, carry-forward balances, and shift anomaly checks.

**Live Sandbox:** [https://enterprise-payroll.vercel.app/](https://enterprise-payroll.vercel.app/) *(click **"Try Interactive Demo"** on the login screen for 1-click anonymous sandbox access with 20 mock employees).*

---

## Problem & Measured Impact

Bi-weekly restaurant payroll often fails standard SMB payroll tools due to interlinked contract rules (tip-pool transfers, guaranteed minimum hour top-ups with deficit carry-forward, split check/cash payouts) and messy daily POS clock-in exports.

* **85% Cycle Time Reduction:** Cut bi-weekly payroll reconciliation from **4+ hours to ~30 minutes** across 20+ shift-based employees.
* **Automated Error Detection:** POS CSV ingestion automatically merges duplicate files, normalizes Unicode accented names (e.g., French characters), and flags potential missed clock-outs (`>9h` shifts) into an audit-tracked validation queue.
* **Zero-Trust AI Guardrails:** LLM outputs never compute dollar totals directly—natural-language rules compile into structured JSON expressions and run inside a deterministic client-side `mathjs` AST evaluator.

---

## Core Architecture & Engineering Highlights

### 1. Deterministic AI Rule Compiler (`api/index.ts`, `server.ts`, `src/lib/payroll-engine.ts`)
* Users describe custom rules in plain English/French (e.g., *"Keep max 20 hours and transfer the rest to employee ID 5"*).
* Backend `/api/generate-rule` authenticates the session (`verifyAuth`), calls Google GenAI (`responseSchema`) to emit structured math expressions (`evaluated_hours`, `evaluated_addons`, `transfer_out_hours`, `transfer_to_id`), and scales weekly figures to bi-weekly cycles.
* Frontend evaluates formulas via `mathjs` (no `eval()`) with hard bounds and a built-in manual formula editor fallback if AI endpoints are unavailable.

### 2. Two-Pass Multi-Contract Engine (`src/lib/payroll-engine.ts`, `src/types.ts`)
* Models **11 contract rule types** (`STANDARD_MAX`, `GUARANTEED_MIN_HOURS`, `FIXED_TOTAL`, `CHECK_PLUS_CASH`, `COST_ALLOCATION_OUT_*`, `COST_ALLOCATION_IN_*`, `NON_PAYROLL_CONTRACTOR`, `AI_COPILOT`, `CUSTOM_LIBRARY_RULE`).
* **Pass 1** resolves individual base earnings, caps, deficits, and transfer outflows; **Pass 2** resolves cross-employee cost allocation inflows and applies cycle caps.

### 3. POS CSV Parser & Validation Audit Trail (`src/lib/pos-parser.ts`)
* Parses multi-format dates, deduplicates overlapping exports, applies NFD Unicode normalization for name matching, flags `>9h` anomalies, and records manager validations/overrides with user email and timestamps.
* Supports one-click **multi-sheet Excel export** (`src/lib/excel-export.ts`, `xlsx`) and CSV export (`src/lib/csv-export.ts`).

### 4. Multi-Tenant Isolation & Ephemeral Demo Sandbox (`api/index.ts`, `src/lib/supabase.ts`)
* Enforces workspace isolation via `company_id` with Supabase PostgreSQL Row Level Security (RLS) and 4-tier RBAC (`OWNER > MANAGER > FINANCE > STAFF`).
* **1-Click Sandbox (`/api/seed-sandbox`, `/api/cron/cleanup-sandbox`):** Uses Supabase Anonymous Auth + RPC seeding (`fn_seed_sandbox`) and stateless session cleanup (`fn_cleanup_sandbox`) after 10 minutes to prevent orphaned database bloat.

---

## Tech Stack

| Layer | Technologies |
| :--- | :--- |
| **Frontend** | React 19, TypeScript, Vite, Tailwind CSS v4, Recharts, Framer Motion, Lucide Icons |
| **Backend API** | Node.js, Express, Vercel Serverless Functions (`api/index.ts`) |
| **Database & Auth** | Supabase (PostgreSQL, Row Level Security, Anonymous & Email Auth, RPC) |
| **AI & Safe Evaluation** | `@google/genai` (Structured JSON output), `mathjs` AST evaluation |
| **Data & Testing** | `xlsx`, custom CSV parser, Puppeteer (`DevTestRunner`) |

---

## Project Structure

```text
├── api/
│   └── index.ts             # Vercel serverless Express app (auth middleware, AI rule compiler, sandbox seed/cleanup)
├── sample_data/
│   └── demo_pos_export.csv  # Synthetic POS shift CSV sample (mock names, includes >9h anomaly case)
├── src/
│   ├── components/          # Dashboard, TimesheetInput, ImportData, EmployeeList, SettingsPanel, Reports, TourOverlay
│   ├── contexts/            # Auth & workspace context providers
│   ├── hooks/               # Payroll state & lifecycle hooks
│   ├── lib/                 # payroll-engine.ts, pos-parser.ts, excel-export.ts, csv-export.ts, supabase.ts
│   ├── App.tsx
│   └── types.ts
├── server.ts                # Local Express + Vite dev server
├── vercel.json              # API & SPA rewrite configuration
└── package.json
```

---

## Local Development

```bash
# 1. Install dependencies
npm install

# 2. Configure environment variables
cp .env.example .env

# 3. Start local Express + Vite server
npm run dev

# Type-check & build production bundle
npm run lint
npm run build
```

---

## Author

**Ba Chanh Truc Le** — Operations & AI Systems Builder  
[LinkedIn](https://www.linkedin.com/in/lebachanhtruc) · Longueuil / Montréal, QC, Canada
