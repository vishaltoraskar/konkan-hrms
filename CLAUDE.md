# CLAUDE.md

## What this project is

An HRMS (HR Management System) for Konkan Coastal Foods, a 180-person food company in Goa with a head office in Panaji and a plant in Verna. It replaces their Excel employee sheet, paper attendance register and WhatsApp leave approvals with one web app covering three modules (Employee Master, Attendance, Leave), a month-end attendance report for the payroll vendor, and an AI "Ask HR" assistant that works by calling the app's own server functions. This is a college assignment; the owner is a beginner, so explain changes in simple words.

## Business rules

All business rules are in `requirements.pdf` (keep a copy in this folder) and must be followed exactly. This covers roles, Indian field validation, day-status rules, the leave policy, agent guardrails and the testing checklist. The leave policy rules are also written out in `RESEARCH.md` under "Leave policy rules we will implement". If a rule is unclear, ask; do not guess. Follow `ARCHITECTURE.md` and `DESIGN.md` once they exist. If the build needs to depart from them, update `ARCHITECTURE.md` and its decision log first.

## Stack

- **Framework:** Next.js (App Router, TypeScript)
- **Database:** Supabase (Postgres, with row-level security on statutory and bank tables)
- **Auth:** Clerk (sign-in only; the user's role is stored in our database, not in Clerk)
- **Email:** Resend
- **Hosting:** Vercel
- **AI agent:** Google Gemini via the Vercel AI SDK (tool calling)

## Folder structure (organised by module)

```
src/
  app/                  # Next.js pages and routes (thin; call into modules)
  modules/
    employees/          # Employee Master: directory, profile, add/edit, masking, audit log
    attendance/         # Check in/out, day status, WFH, holidays, regularisation, team view
    leave/              # Leave types, balances, apply, approve, cancel, leave calendar, emails
    reports/            # Month-end summary and CSV download
    agent/              # Ask HR agent: tools, system prompt, agent_log
  lib/                  # Shared helpers: Supabase client, auth/role checks, IST date helpers, validation
  components/           # Shared UI components (button, card, table, form field, status badge)
supabase/
  migrations/           # Database tables and constraints
  seed/                 # Seed script (2 locations, 4 departments, 20 employees, holidays, balances)
tests/                  # Automated tests, grouped by module
```

Each module keeps its own server functions, validation and UI pieces together. The agent's tools reuse the server functions from the other modules; they do not duplicate logic.

## Rules

- **Time zone:** all times are stored and shown in IST (`Asia/Kolkata`), whatever the server's time zone. Use the shared IST helpers in `src/lib`; never use the server's local time.
- **Secrets:** never commit secrets, API keys or `.env` / `.env.local` files. Only `.env.example` with blank values is committed.
- **Role checks:** always on the server, on every page, server action and API route. Hiding a button is not a role check. Only the HR Admin flag is stored; Manager is derived (anyone who is `reporting_manager_id` of at least one active employee).
- **Sensitive data:** Aadhaar, PAN and bank details are always masked (PAN `ABCDE****F`, Aadhaar `XXXX XXXX 1234`, bank account last 4 digits) on every screen and in every CSV. Only HR can Reveal, and every reveal is logged. These fields are never sent to the AI model, not even masked.
- **Database:** never run a database reset, drop tables or delete data without asking the owner first.
- **Build order:** the owner has chosen to build first and write `ARCHITECTURE.md` and `DESIGN.md` afterwards (a departure from the PDF's order). Do not refuse code for lack of sign-off, but work one step at a time and stop when the owner says to.
