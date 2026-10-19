# Tasks Document

Requirement references use `requirements.md` numbering (`R1`–`R11`) plus the PM's AC IDs. Paths follow the developer
project template (`koldovsky/2026-quitcode-06-project-template`).

## 1. Database

- [ ] 1.1. Switch app roles to employee / manager / admin
  - File: supabase/migrations/20261020090000_app_roles.sql
  - Drop `profiles_role_check`, move `member` → `employee`, set default `employee`, add the new check — in that order
    (recipe in the header of the first migration).
  - Purpose: give the portal its three roles without breaking `npm run users:demo`.
  - _Leverage: supabase/migrations/20261012000000_profiles_and_roles.sql, private.current_app_role()_
  - _Requirements: R5, R7, R11_
  - _Prompt: Role: Postgres/Supabase developer | Task: Write a migration that replaces roles member/admin with employee/manager/admin on public.profiles, following the 3-step order in the header of 20261012000000_profiles_and_roles.sql | Restrictions: Do not edit the existing migration; keep column-level grants unchanged; the timestamp must be newer than every migration on main | Success: supabase db reset succeeds, npm run users:demo creates users with role employee, and the check rejects 'member'_

- [ ] 1.2. Create organisation and schedule tables
  - File: supabase/migrations/20261020090100_positions_and_schedules.sql
  - Tables `divisions`, `positions` (one main per employee), `position_managers`, `schedule_blocks`, `app_settings`
    (`time_zone = Europe/Kyiv`); RLS: employees read their own positions and blocks, managers those they manage, admin
    all; no insert/update for authenticated.
  - Purpose: the data that determines working days and which manager receives which record.
  - _Leverage: RLS conventions from 20261012000000_profiles_and_roles.sql_
  - _Requirements: R1, R2, R5_
  - _Prompt: Role: Postgres/Supabase developer | Task: Create the tables divisions, positions, position_managers, schedule_blocks and app_settings exactly as in design.md "Data Models", with RLS enabled, revoke-all then explicit grants, one policy per operation, and a private.is_manager_of(position_id) security-definer helper | Restrictions: Helpers only in schema private with search_path = ''; auth.uid() only as (select auth.uid()); no salary columns | Success: Migration applies; a manager can select only positions in position_managers; an employee only their own; admin all_

- [ ] 1.3. Create leave tables, overlap constraint and events
  - File: supabase/migrations/20261020090200_leave_records.sql
  - `leave_types` (seeded sick/personal 18.00, unpaid/family null), `balance_carryovers`, `leave_requests`,
    `leave_records` with the `btree_gist` exclusion constraint for statuses awaiting/returned/approved,
    `record_events`; select policies only (writes via RPC).
  - Purpose: store requests, records and their history with overlap protection in the database.
  - _Leverage: design.md Data Models_
  - _Requirements: R1, R2 (AC-LEAVE-13), R6 (AC-DECIDE-15), R9_
  - _Prompt: Role: Postgres/Supabase developer | Task: Create the leave tables from design.md including the partial exclusion constraint on (position_id, tstzrange(start_at, end_at)) and select policies so an employee sees own records, a responsible manager sees records of managed positions, admin sees all | Restrictions: No insert/update/delete grants to authenticated on these tables; enable btree_gist with create extension if not exists | Success: Inserting two overlapping awaiting records for one position fails with 23P01; a rejected record does not block; RLS returns only allowed rows for each demo user_

- [ ] 1.4. Seed test data
  - Files: supabase/seed.sql, supabase/demo-users.json
  - Demo users teacher1, teacher2, manager1, manager2, admin `@example.test`; divisions "Elementary school", "High
    school"; teacher1 main "Elementary school / Teacher" Mon–Fri 08:30–15:30 (Tue also as 08:30–12:00 + 12:30–15:30
    variant on a second test position) and additional "High school / Math teacher" Mon 15:45–17:00; manager1 → main,
    manager2 → additional.
  - Purpose: data the ACs refer to, so every AC can be reproduced locally.
  - _Leverage: scripts/create-demo-users.mjs, supabase/demo-users.json_
  - _Requirements: All_
  - _Prompt: Role: Developer preparing test fixtures | Task: Add seed rows and demo users matching the examples in the AC texts (dates in November 2026, schedules, managers, carry-over 0) | Restrictions: Only synthetic names and @example.test emails; never load seed data in production | Success: supabase db reset plus npm run users:demo produce users and data with which AC-LEAVE-01, -02, -15 and AC-DECIDE-01 can be run by hand_

## 2. Domain (pure functions)

- [ ] 2.1. Schedule and interval functions
  - File: src/lib/leave/schedule.ts
  - `blocksFor`, `validateInterval`, `minutesInBlocks`, 5-minute grid, block start inclusive / end exclusive, intervals
    spanning a gap allowed.
  - Purpose: one source of truth for working-hour rules used by the form and mirrored in SQL.
  - _Leverage: none_
  - _Requirements: R1 (AC-LEAVE-04, -15), R2 (AC-LEAVE-06..09)_
  - _Prompt: Role: TypeScript developer | Task: Implement pure functions from design.md Component 1 with exported types Block, ErrorCode | Restrictions: No I/O, no Date timezone tricks — work with local wall-clock "HH:MM" strings plus ISO dates; no dependencies | Success: Functions return the error codes for AC-LEAVE-07, -08, -09, 90 minutes for AC-LEAVE-15, and null for valid input_

- [ ] 2.2. Duration, school year and split functions
  - Files: src/lib/leave/duration.ts, src/lib/leave/balance.ts
  - `dayShare` (round half up, 2 dp, cap 1.00), `schoolYearOf` (1 Sep–31 Aug), `suggestSplit`.
  - Purpose: shared arithmetic for previews and warnings.
  - _Leverage: none_
  - _Requirements: R9 (AC-BALANCE-03, -05, -06, -11), R10 (AC-BALANCE-08, -15)_
  - _Prompt: Role: TypeScript developer | Task: Implement dayShare, schoolYearOf and suggestSplit as in design.md Component 2 using integer minutes and hundredths to avoid float error | Restrictions: Pure functions only; no floating-point equality in results | Success: dayShare(150, 420) = 0.36, dayShare(75, 495) = 0.15, suggestSplit(1.00, 0.50) = {paid: 0.50, unpaid: 0.50}, suggestSplit(0.36, 0.20) = {paid: 0.20, unpaid: 0.16}_

- [ ] 2.3. Error-code message map
  - File: src/lib/leave/messages.ts
  - Every code from design.md Component 3 mapped to the exact AC text with parameters.
  - Purpose: identical user-facing texts everywhere.
  - _Leverage: none_
  - _Requirements: R2, R6, R7, R8, R10_
  - _Prompt: Role: TypeScript developer | Task: Create messageFor(code, params) returning the exact English messages quoted in the ACs | Restrictions: Texts must match the AC strings character for character; unknown codes return a generic message and are logged | Success: Each AC message can be produced from its code_

- [ ] 2.4. Unit tests for domain functions
  - Files: tests/leave/schedule.test.ts, tests/leave/duration.test.ts, tests/leave/balance.test.ts,
    tests/leave/messages.test.ts
  - Purpose: lock the arithmetic and validation rules; titles carry AC IDs.
  - _Leverage: vitest.config.mts_
  - _Requirements: AC-LEAVE-04, -06, -07, -08, -09, -15; AC-BALANCE-03, -05, -06, -08, -11, -15_
  - _Prompt: Role: QA engineer | Task: Write Vitest unit tests, one it() per AC listed, titled "AC-XXX-NN: …" | Restrictions: No database, no network; test both success and refusal paths | Success: npm test passes and npm run ac:coverage finds a test for each listed AC_

## 3. Database functions (RPC)

- [ ] 3.1. `submit_leave_request`
  - File: supabase/migrations/20261020090300_rpc_submit.sql
  - Validate every row (schedule, block edges, end > start, at least one position, relationship for family), compute
    `minutes` and `day_total_minutes`, insert request + records + events in one transaction, map 23P01 to
    `LEAVE_OVERLAP`, return the over-balance warning.
  - Purpose: the only write path for new leave, safe for direct API calls.
  - _Leverage: tables from 1.2–1.3, rules from 2.1–2.2_
  - _Requirements: R1, R2, R10 (AC-BALANCE-08, -15)_
  - _Prompt: Role: Postgres developer | Task: Implement private.submit_leave_request(payload jsonb) and a public wrapper granted to authenticated, raising error codes from design.md | Restrictions: security definer in private with search_path = ''; employee_id always = (select auth.uid()); all-or-nothing | Success: AC-LEAVE-01..05, -10, -12, -13, -15, -16 behave as written when called directly through the Data API_

- [ ] 3.2. `decide_record`
  - File: supabase/migrations/20261020090400_rpc_decide.sql
  - Permission check (admin, or responsible manager who is not the record's employee), `for update` lock, status
    `awaiting` required, comment for return, balance recomputed under lock, split required when exceeded, snapshot
    `paid_days`/`unpaid_days`, event row.
  - Purpose: every manager/admin decision with its rules in one place.
  - _Leverage: private.is_manager_of from 1.2, day-share rule in design.md_
  - _Requirements: R6, R7, R10 (AC-BALANCE-09, -10, -14, -16)_
  - _Prompt: Role: Postgres developer | Task: Implement private.decide_record(record_id uuid, decision text, comment text, accept_split boolean) per design.md Component 6 | Restrictions: Never trust client-sent balance numbers; raise DECIDE_FORBIDDEN, DECIDE_OWN_RECORD, DECIDE_ALREADY_DECIDED, DECIDE_COMMENT_REQUIRED, BALANCE_SPLIT_REQUIRED | Success: AC-DECIDE-02..05, -07..-10, -18, -19 and AC-BALANCE-10, -14, -16 pass as integration tests_

- [ ] 3.3. Employee record actions
  - File: supabase/migrations/20261020090500_rpc_own_records.sql
  - `update_own_record` (awaiting/returned only; re-validates and keeps or sets `awaiting`), `resubmit_record`,
    `cancel_own_record` (awaiting/returned → cancelled).
  - Purpose: employee corrections without touching decided records.
  - _Leverage: validation from 3.1_
  - _Requirements: R8_
  - _Prompt: Role: Postgres developer | Task: Implement the three RPCs with ownership checks and RECORD_LOCKED for other statuses | Restrictions: Only the record's employee may call them; reuse the 3.1 validation function instead of duplicating it | Success: AC-DECIDE-06, -12, -13, -16, -17 hold through the Data API_

- [ ] 3.4. Read views
  - File: supabase/migrations/20261020090600_views.sql
  - `my_records`, `manager_queue` (awaiting + history), `record_history`, `leave_balances` (granted, carried_over, used,
    remaining per type and school year) with `security_invoker = true`.
  - Purpose: role-filtered reads for all pages.
  - _Leverage: RLS from 1.2–1.3_
  - _Requirements: R4, R5, R6 (AC-DECIDE-15), R9, R11_
  - _Prompt: Role: Postgres developer | Task: Create the views from design.md with security_invoker so RLS applies; leave_balances counts only approved paid_days and adds carry-over | Restrictions: No security definer views; no salary data | Success: Each demo user sees exactly the rows the ACs allow; AC-BALANCE-01, -02, -04, -07, -11 values are produced by the view_

- [ ] 3.5. Database integration tests
  - Files: tests/db/submit.test.ts, tests/db/decide.test.ts, tests/db/own-records.test.ts, tests/db/balances.test.ts,
    tests/db/rls.test.ts, tests/db/helpers.ts
  - Purpose: prove the rules hold without the UI, for each role.
  - _Leverage: supabase-js, demo users, seed from 1.4_
  - _Requirements: R1–R11 (all ACs not covered in 2.4 or E2E)_
  - _Prompt: Role: QA engineer | Task: Write Vitest integration tests that sign in as each demo user against local Supabase and call RPCs/views, one it() per AC titled with its ID | Restrictions: Reset data between tests; never use the service-role key for the actor under test | Success: Every AC from the integration list in design.md has a passing test_

## 4. Server actions

- [ ] 4.1. Request actions
  - File: src/app/(app)/requests/actions.ts
  - `submitRequest`, `updateOwnRecord`, `resubmitRecord`, `cancelOwnRecord`: call RPCs, translate error codes via
    `messageFor`, revalidate paths.
  - _Leverage: src/lib/supabase/server.ts, src/lib/leave/messages.ts_
  - _Requirements: R1, R2, R8, R10_
  - _Prompt: Role: Next.js developer | Task: Implement server actions returning {ok, ...} or {ok: false, message} | Restrictions: No direct table writes; validate input shape before calling RPC | Success: Actions surface the exact AC messages and never throw raw database errors to the UI_

- [ ] 4.2. Decision action
  - File: src/app/(app)/records/[id]/actions.ts
  - `decideRecord(recordId, decision, comment?, acceptSplit?)`.
  - _Leverage: same as 4.1_
  - _Requirements: R6, R7, R10_
  - _Prompt: Role: Next.js developer | Task: Implement decideRecord calling the decide_record RPC and mapping codes to messages | Restrictions: Never decide on the client; refresh the record after DECIDE_ALREADY_DECIDED | Success: AC-DECIDE-07 message shown and card refreshed_

## 5. Frontend

- [ ] 5.1. New request page
  - Files: src/app/(app)/requests/new/page.tsx, src/app/(app)/requests/new/RequestForm.tsx,
    src/app/(app)/requests/new/SummaryDialog.tsx
  - Date picker (only working days), per-position rows (include, whole day / hours on a 5-minute grid), leave type,
    relationship for family celebration, summary dialog with Edit / Confirm, warnings.
  - _Leverage: 2.1, 2.2, 4.1_
  - _Requirements: R1, R2, R3, R10 (AC-BALANCE-08, -15)_
  - _Prompt: Role: React/Next.js developer | Task: Build the form and summary dialog; show pure-function validation instantly and server messages on submit | Restrictions: Nothing is saved before Confirm; form starts empty; desktop-first; times displayed in Europe/Kyiv | Success: AC-LEAVE-06, -11 visible in the browser; submit creates records as in AC-LEAVE-01..05_

- [ ] 5.2. My requests page
  - Files: src/app/(app)/requests/page.tsx, src/app/(app)/requests/RecordRow.tsx
  - List with date, hours, position, type, status, manager comment; edit / resubmit / cancel where allowed.
  - _Leverage: view my_records, 4.1_
  - _Requirements: R4, R8_
  - _Prompt: Role: React/Next.js developer | Task: Render own records with allowed actions by status | Restrictions: Paginate server-side (50); hide actions not allowed by status but rely on the server for enforcement | Success: AC-LEAVE-14 and AC-DECIDE-05, -06, -13, -16, -17 work through the UI_

- [ ] 5.3. Manager workspace and record card
  - Files: src/app/(app)/manager/page.tsx, src/app/(app)/records/[id]/page.tsx, src/app/(app)/records/[id]/DecisionPanel.tsx
  - "Awaiting decision" (default) and "History" tabs, empty state, record card with balance warning, split suggestion,
    history lines, approve / reject / return (comment required).
  - _Leverage: views manager_queue, record_history, leave_balances; 4.2_
  - _Requirements: R5, R6, R7, R10_
  - _Prompt: Role: React/Next.js developer | Task: Build the manager list and card | Restrictions: Admin sees all records; manager sees only managed positions (RLS); show "Your own record is decided by an administrator" from the server | Success: AC-DECIDE-01, -11, -14, -15 and AC-BALANCE-09 visible as written_

- [ ] 5.4. Balances page
  - File: src/app/(app)/balances/page.tsx
  - Own balances for the current school year; managers/admin can open an employee's balances they are allowed to see.
  - _Leverage: view leave_balances_
  - _Requirements: R9, R11_
  - _Prompt: Role: React/Next.js developer | Task: Show granted, carried over, used, remaining with 2 decimals; usage only for unlimited types | Restrictions: No client-side balance computation | Success: AC-BALANCE-01 and -11 render exactly; AC-BALANCE-12, -13 show nothing for unauthorised viewers_

## 6. Integration, E2E and wrap-up

- [ ] 6.1. End-to-end tests
  - Files: tests/e2e/leave-flow.spec.ts, playwright.config.ts
  - Summary dialog does not save (AC-LEAVE-11), Saturday refused (AC-LEAVE-06), submit → approve → balance happy path.
  - _Leverage: demo users, seed_
  - _Requirements: R1, R3, R6, R9_
  - _Prompt: Role: QA automation engineer | Task: Add Playwright tests titled with AC IDs | Restrictions: Only if the course allows adding Playwright; otherwise record these ACs as manually verified in docs/conformance.md | Success: Tests pass locally against npm run dev with a reset database_

- [ ] 6.2. Conformance table and cleanup
  - Files: docs/conformance.md, docs/spec-reading.md
  - One row per AC (51) with status, code link and test link; record own working assumptions; remove dead code; run
    `npm run lint`, `typecheck`, `test`, `ac:coverage`, `spec:check`.
  - _Leverage: scripts/check-ac-coverage.mjs, scripts/check-spec.mjs_
  - _Requirements: All_
  - _Prompt: Role: Senior developer closing a slice | Task: Fill docs/conformance.md for all AC IDs and make every check script pass | Restrictions: Do not rename AC IDs; mark anything not done as deferred with a reason | Success: All npm scripts green; every AC has a row with code and test links_
