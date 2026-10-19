Spec 1 of 3 — leave request. Depends on: none. Used by: spec-2-leave-decision, spec-3-leave-balance.

# Tasks Document

Requirement references use this spec's `requirements.md` numbering (`1.1` = Requirement 1, criterion 1), plus the PM's
AC IDs and the business rules `BR-n`. Component numbers refer to `design.md`. Paths follow the developer project
template (`koldovsky/2026-quitcode-06-project-template`). Migrations use timestamps `20261020090000`–`20261020090400`;
spec-2 and spec-3 continue after them.

## 1. Database foundation

- [ ] 1.1. Switch app roles to employee / manager / admin
  - File: supabase/migrations/20261020090000_app_roles.sql
  - Drop `profiles_role_check`, move `member` → `employee`, set default `employee`, add the new check — in that order.
  - Purpose: give the portal its three roles without breaking `npm run users:demo` (Component 1).
  - _Leverage: supabase/migrations/20261012000000_profiles_and_roles.sql, private.current_app_role()_
  - _Requirements: 4.1 (AC-LEAVE-14), BR-11_
  - _Prompt: Role: Postgres/Supabase developer | Task: Write a migration that replaces roles member/admin with employee/manager/admin on public.profiles, following the 3-step order in the header of 20261012000000_profiles_and_roles.sql, as described in design.md Component 1 | Restrictions: Do not edit the existing migration; keep column-level grants unchanged; the timestamp must be newer than every migration on main | Success: supabase db reset succeeds, npm run users:demo creates users with role employee, and the check rejects 'member'_

- [ ] 1.2. Create organisation and schedule tables with select RLS
  - File: supabase/migrations/20261020090100_positions_and_schedules.sql
  - Tables `divisions`, `positions` (one main per employee), `position_managers`, `schedule_blocks` (5-minute checks),
    `app_settings` (`time_zone = Europe/Kyiv`); helpers `private.is_manager_of(position_id)` and `private.is_admin()`;
    select policies only, as in design.md Component 2.
  - Purpose: the data that determines working days, and which manager receives which record (shared with specs 2–3).
  - _Leverage: RLS conventions from supabase/migrations/20261012000000_profiles_and_roles.sql_
  - _Requirements: 1.1–1.7, 2.1–2.4 (schedules), BR-1, BR-2, BR-6_
  - _Prompt: Role: Postgres/Supabase developer | Task: Create divisions, positions, position_managers, schedule_blocks and app_settings exactly as in design.md "Data Models", with RLS enabled, revoke-all then explicit select grants, one policy per operation to authenticated, and security-definer helpers private.is_manager_of(p_position_id uuid) and private.is_admin() | Restrictions: Helpers only in schema private with set search_path = ''; auth.uid() only as (select auth.uid()); no insert/update/delete grants to authenticated; no salary columns; avoid policy recursion between positions and position_managers | Success: Migration applies; an employee selects only own positions and blocks; a manager additionally those in position_managers; admin all_

- [ ] 1.3. Create leave tables, overlap constraint and events
  - File: supabase/migrations/20261020090200_leave_records.sql
  - `btree_gist`; `leave_types` seeded (sick, personal, unpaid, family); `leave_requests` (relationship check);
    `leave_records` (minutes, day_total_minutes, status, manager_comment, decided_by, decided_at — no paid/unpaid days,
    spec-3 adds them) with the partial exclusion constraint `leave_records_no_overlap`; `record_events`; select
    policies only.
  - Purpose: store requests, records and their history with overlap protection in the database (Component 3).
  - _Leverage: design.md Data Models, private.is_manager_of from 1.2_
  - _Requirements: 1.5, 1.7 (AC-LEAVE-05, -16), 2.6 (AC-LEAVE-12), 2.7 (AC-LEAVE-13), 4.1 (AC-LEAVE-14), BR-8, BR-9, BR-12_
  - _Prompt: Role: Postgres/Supabase developer | Task: Create leave_types, leave_requests, leave_records and record_events from design.md, including EXCLUDE USING gist (position_id WITH =, tstzrange(start_at, end_at, '[)') WITH &&) WHERE status IN ('awaiting','returned','approved'), and select policies so an employee sees own rows, a responsible manager sees records of managed positions, admin sees all | Restrictions: No insert/update/delete grants to authenticated; create extension if not exists btree_gist; do not add paid_days, unpaid_days, annual_limit or balance tables (spec-3) | Success: Two overlapping awaiting records for one position fail with 23P01; adjacent records (10:00–11:00, 11:00–12:00) succeed; a rejected record does not block; each demo user selects only allowed rows_

- [ ] 1.4. Seed demo users and leave test data
  - Files: supabase/seed.sql, supabase/demo-users.json, scripts/seed-leave-demo.mjs
  - Divisions in `seed.sql`; demo users teacher1, teacher2, manager1, manager2, admin `@example.test` with their roles;
    idempotent script that creates positions, responsible managers and schedule blocks from the table in design.md
    Component 11, run with `node scripts/seed-leave-demo.mjs`.
  - Purpose: data the ACs refer to, so every AC can be reproduced locally and by tests.
  - _Leverage: scripts/create-demo-users.mjs, supabase/demo-users.json_
  - _Requirements: All_
  - _Prompt: Role: Developer preparing test fixtures | Task: Add divisions to seed.sql, demo users to demo-users.json, and scripts/seed-leave-demo.mjs that resolves users by email and upserts positions, position_managers and schedule_blocks exactly as in design.md Component 11 (teacher1 Mon–Fri 08:30–15:30 + Mon 15:45–17:00; teacher2 08:30–13:00 and Tue 08:30–12:00 + 12:30–15:30) | Restrictions: Only synthetic names and @example.test emails; the script refuses to run unless SUPABASE_URL points to localhost; no leave records seeded; re-running must not duplicate rows | Success: supabase db reset, npm run users:demo and node scripts/seed-leave-demo.mjs produce data with which AC-LEAVE-01, -02, -07, -15 can be run by hand_

## 2. Domain (pure functions)

- [ ] 2.1. Schedule and interval functions
  - File: src/lib/leave/schedule.ts
  - `blocksFor`, `wholeDay`, `validateInterval`, `minutesInBlocks`, `formatDuration`; 5-minute grid, block start
    inclusive / end exclusive, intervals spanning a gap allowed; check order from design.md Component 4.
  - Purpose: one source of truth for working-hour rules used by the form and mirrored in SQL.
  - _Leverage: none (no existing domain code)_
  - _Requirements: 1.1, 1.2, 1.4 (AC-LEAVE-04), 1.6 (AC-LEAVE-15), 2.1–2.4 (AC-LEAVE-06..09), BR-2..BR-5_
  - _Prompt: Role: TypeScript developer | Task: Implement the pure functions and exported types Block and ScheduleBlock from design.md Component 4, returning { code, params } objects with LEAVE_* codes | Restrictions: No I/O; no Date time-zone conversion — work with "HH:MM" wall-clock strings and ISO dates (weekday via UTC date arithmetic); no new dependencies | Success: validateInterval returns LEAVE_OUTSIDE_SCHEDULE {blocks: "08:30–13:00"} for 08:00–10:00, LEAVE_START_AT_BLOCK_END {time: "13:00"} for start 13:00, LEAVE_END_BEFORE_START for 11:00–10:00; minutesInBlocks gives 150 and 90 for AC-LEAVE-04 and -15; formatDuration(150) = "2 h 30 min"_

- [ ] 2.2. Error-code message map
  - File: src/lib/leave/messages.ts
  - `LeaveErrorCode`, `messageFor`, `parseRpcError`; only `LEAVE_*` codes from design.md Component 5 (specs 2 and 3 add
    their own codes to this file).
  - Purpose: identical user-facing texts in the form and for server errors.
  - _Leverage: none_
  - _Requirements: 1.7 (AC-LEAVE-16), 2.1–2.7 (AC-LEAVE-06..10, -12, -13)_
  - _Prompt: Role: TypeScript developer | Task: Create messageFor(code, params) and parseRpcError(error) returning the exact English messages from the table in design.md Component 5 | Restrictions: Texts must match the AC strings character for character (en dash in time ranges); unknown codes return the LEAVE_INVALID_INPUT text; structure the map so other specs can add DECIDE_*, RECORD_* and BALANCE_* entries without editing existing ones | Success: Every AC message in Requirement 2 and AC-LEAVE-16 can be produced from its code and params_

- [ ] 2.3. Unit tests for domain functions
  - Files: tests/leave/schedule.test.ts, tests/leave/messages.test.ts
  - One `it()` per pure AC, titled "AC-LEAVE-NN: …", plus BR edge cases (12:55 start allowed, 12:58 refused, end equal
    to block end allowed, whole day across a gap = sum of blocks).
  - Purpose: lock the working-hour rules and texts before the SQL mirrors them.
  - _Leverage: vitest.config.mts_
  - _Requirements: 1.4, 1.6, 2.1–2.7 (AC-LEAVE-04, -06, -07, -08, -09, -15; texts for -10, -12, -13), BR-2, BR-3, BR-5_
  - _Prompt: Role: QA engineer | Task: Write Vitest unit tests for schedule.ts and messages.ts, one it() per listed AC titled with its ID, covering both valid and refused inputs | Restrictions: No database, no network; no snapshot tests for messages — assert exact strings | Success: npm test passes and npm run ac:coverage finds a test for each listed AC_

## 3. Database functions and views

- [ ] 3.1. `submit_leave_request` RPC
  - File: supabase/migrations/20261020090300_rpc_submit.sql
  - `private.position_blocks(position_id, date)`, `private.submit_leave_request(payload jsonb)` and the
    `public.submit_leave_request` wrapper; steps 1–10 from design.md Component 6; 23P01 → `LEAVE_OVERLAP` with the
    conflicting record's Kyiv date and times; returns `{request_id, record_ids}` with no balance warning (spec-3
    extends it).
  - Purpose: the only write path for new leave, all-or-nothing and safe for direct API calls.
  - _Leverage: tables from 1.2–1.3, rules from 2.1_
  - _Requirements: 1.1–1.7 (AC-LEAVE-01..05, -15, -16), 2.1–2.7 (AC-LEAVE-06..10, -12, -13), BR-1..BR-11_
  - _Prompt: Role: Postgres developer | Task: Implement the functions in design.md Component 6, raising errors as raise exception using message = '<CODE>', detail = '<json params>' | Restrictions: security definer only in private with set search_path = ''; employee_id always (select auth.uid()); every position must belong to that employee and be active on the date; whole transaction rolls back on any error; grant execute on the public wrapper to authenticated only | Success: Called directly through the Data API, AC-LEAVE-01..05, -10, -12, -13, -15 and -16 behave as written; a rejected record does not block; a past date is accepted; someone else's position is refused_

- [ ] 3.2. `my_records` view
  - File: supabase/migrations/20261020090400_view_my_records.sql
  - `security_invoker = true`, explicit `employee_id = (select auth.uid())`, columns from design.md Component 7
    (including `manager_comment` for spec-2), Kyiv `HH:MM` times.
  - Purpose: the read model for "My requests".
  - _Leverage: RLS from 1.2–1.3_
  - _Requirements: 4.1 (AC-LEAVE-14), BR-1_
  - _Prompt: Role: Postgres developer | Task: Create public.my_records as specified, joining positions, divisions and leave_types; grant select to authenticated | Restrictions: No security definer view; filter on the signed-in user even though RLS would allow managers/admins more rows; keep column order stable so spec-2 can append columns | Success: teacher1 sees exactly own records; manager1 sees only own records here although leave_records shows managed records too; times show in Europe/Kyiv_

- [ ] 3.3. Test helpers, schema and RLS integration tests
  - Files: tests/db/helpers.ts, tests/db/schema.test.ts, tests/db/rls.test.ts
  - `helpers.ts`: sign in as a demo user with the anon key, service-role client for cleanup only; schema tests (role
    check, exclusion constraint, no direct insert for `authenticated`); RLS tests (`my_records` per user incl.
    manager1, positions and schedules per role).
  - Purpose: prove the foundation and visibility rules hold without the UI, for each role.
  - _Leverage: @supabase/supabase-js, demo users and seed from 1.4, vitest.config.mts_
  - _Requirements: 2.7 (AC-LEAVE-13 constraint), 4.1 (AC-LEAVE-14), BR-9, BR-11_
  - _Prompt: Role: QA engineer | Task: Write the helpers and the schema/RLS Vitest integration tests against local Supabase listed in design.md "Integration Testing", titles carrying AC IDs | Restrictions: Never use the service-role key for the actor under test; delete leave data between tests; tests must not depend on order | Success: AC-LEAVE-14 passes for teacher1 (2 own records of 5) and for manager1 (only own records in my_records); direct inserts into leave_records are refused_

- [ ] 3.4. Submit RPC integration tests
  - File: tests/db/submit.test.ts
  - One `it()` per AC through `submit_leave_request` as a signed-in demo user; service role only to set a record to
    `rejected` in the BR-9 setup.
  - Purpose: prove the write path and all-or-nothing behaviour without the UI.
  - _Leverage: tests/db/helpers.ts from 3.3_
  - _Requirements: 1.1–1.7, 2.5–2.7 (AC-LEAVE-01..05, -10, -12, -13, -15, -16), BR-7, BR-9, BR-10, BR-11_
  - _Prompt: Role: QA engineer | Task: Write Vitest integration tests for the submit RPC following design.md "Integration Testing" | Restrictions: Assert both the error code/message and that no rows were written; use dates and schedules exactly as in the AC texts | Success: All listed ACs pass; AC-LEAVE-16 proves no record exists for 2026-11-02 or 2026-11-03 from the failed request_

- [ ] 3.5. Regenerate database types
  - File: src/lib/supabase/database.types.ts (path as used by the template's `npm run db:types`)
  - Purpose: typed RPC payload and `my_records` rows for the actions and pages.
  - _Leverage: npm run db:types_
  - _Requirements: 1.1, 4.1_
  - _Prompt: Role: TypeScript developer | Task: Run npm run db:types after migrations 1.1–3.2 and keep the regenerated file in the working tree | Restrictions: Do not hand-edit the generated file | Success: npm run typecheck passes with supabase.rpc('submit_leave_request') and from('my_records') typed_

## 4. Server action

- [ ] 4.1. `submitRequest` action
  - File: src/app/(app)/requests/actions.ts
  - Validate input shape, call the RPC, map errors with `parseRpcError` + `messageFor`, `revalidatePath('/requests')`;
    return `{ ok: true, requestId }` or `{ ok: false, message, rowKey? }`.
  - Purpose: the server boundary between the form and the database (Component 8).
  - _Leverage: src/lib/supabase/server.ts, src/lib/leave/messages.ts_
  - _Requirements: 1.1–1.7, 2.1–2.7_
  - _Prompt: Role: Next.js developer | Task: Implement the "use server" action submitRequest(input: SubmitRequestInput) from design.md Component 8 | Restrictions: No direct table writes; never return raw database error text; log unknown errors on the server; leave room in the file for spec-2 actions | Success: The action surfaces the exact AC messages for each LEAVE_* code and returns the request id on success_

## 5. Frontend

- [ ] 5.1. New request form and position rows
  - Files: src/app/(app)/requests/new/page.tsx, src/app/(app)/requests/new/RequestForm.tsx,
    src/app/(app)/requests/new/PositionRow.tsx
  - Server page loads own positions, schedule blocks and leave types; date picker allows only dates with working hours;
    per date, one row per scheduled position (include, whole day / hours on a 5-minute grid); leave type; relationship
    select shown for Family celebration; instant validation with `schedule.ts`.
  - Purpose: let the employee compose a multi-date, multi-position request (Component 9).
  - _Leverage: src/lib/leave/schedule.ts, src/lib/leave/messages.ts_
  - _Requirements: 1.1–1.6, 2.1–2.6 (AC-LEAVE-01..10, -12, -15)_
  - _Prompt: Role: React/Next.js developer | Task: Build the page and client components from design.md Component 9 with Tailwind | Restrictions: Form state lives in RequestForm; times displayed in Europe/Kyiv; desktop-first; client validation is for feedback only — the server decides | Success: Saturday 2026-11-07 cannot be added and shows "You have no working hours on this day"; row errors show the exact AC texts; durations shown as "2 h 30 min"_

- [ ] 5.2. Summary dialog and submit flow
  - Files: src/app/(app)/requests/new/SummaryDialog.tsx, src/app/(app)/requests/new/RequestForm.tsx
  - Dialog lists dates, hours, durations, positions and type; "Back to editing" closes it with state intact; "Confirm"
    calls `submitRequest`, shows server errors in the form, redirects to `/requests` on success.
  - Purpose: nothing is saved before the employee confirms (Requirement 3).
  - _Leverage: submitRequest from 4.1_
  - _Requirements: 3.1 (AC-LEAVE-11), 1.7 (AC-LEAVE-16), 2.7 (AC-LEAVE-13)_
  - _Prompt: Role: React/Next.js developer | Task: Build SummaryDialog and wire Confirm to submitRequest with a pending state | Restrictions: No server call before Confirm; disable Confirm while pending to avoid double submission; keep all input after a server error | Success: Back to editing keeps dates and hours and creates no record; an overlap shows "You already have a request for this time: …" and keeps the form_

- [ ] 5.3. My requests page (read-only)
  - Files: src/app/(app)/requests/page.tsx, src/app/(app)/requests/RecordRow.tsx
  - Reads `my_records`, paginated by 50 via `?page=`; columns date, hours, duration, position, leave type, status
    label; empty state with a link to "New request"; navigation link to both pages.
  - Purpose: the employee sees their own records and statuses (Component 10).
  - _Leverage: view my_records from 3.2, src/lib/supabase/server.ts_
  - _Requirements: 4.1 (AC-LEAVE-14)_
  - _Prompt: Role: React/Next.js developer | Task: Render the read-only list of own records as in design.md Component 10 | Restrictions: No edit/resubmit/cancel controls and no manager comment yet (spec-2 adds them to RecordRow); server-side pagination only | Success: teacher1 sees exactly their 2 records with date, hours, position, leave type and status; teacher2's records never appear_

## 6. Integration, E2E and wrap-up

- [ ] 6.1. End-to-end tests
  - Files: tests/e2e/leave-request.spec.ts, playwright.config.ts
  - AC-LEAVE-11 (Back to editing keeps input, no record), AC-LEAVE-06 (Saturday refused in the picker), AC-LEAVE-01
    happy path ending in "My requests".
  - Purpose: cover the behaviours that exist only in the browser.
  - _Leverage: demo users and seed from 1.4, tests/db/helpers.ts for cleanup_
  - _Requirements: 1.1 (AC-LEAVE-01), 2.1 (AC-LEAVE-06), 3.1 (AC-LEAVE-11), 4.1 (AC-LEAVE-14)_
  - _Prompt: Role: QA automation engineer | Task: Add Playwright tests titled with AC IDs that sign in as teacher1 through /login | Restrictions: Only if the course allows adding Playwright; otherwise record these ACs as manually verified in docs/conformance.md with the steps; reset leave data before each test | Success: Tests pass locally against npm run dev with a reset and seeded database_

- [ ] 6.2. Conformance table and cleanup
  - Files: docs/conformance.md, docs/spec-reading.md
  - One row per AC (AC-LEAVE-01..16) with status, code link and test link; note the open seams for spec-2 and spec-3;
    remove dead code and debug logging; run `npm run lint`, `typecheck`, `test`, `ac:coverage`, `spec:check`.
  - Purpose: close the spec with every AC traceable to code and tests.
  - _Leverage: scripts/check-ac-coverage.mjs, scripts/check-spec.mjs, scripts/check-migration-order.mjs_
  - _Requirements: All_
  - _Prompt: Role: Senior developer closing a spec | Task: Fill docs/conformance.md for all 16 AC-LEAVE IDs and make every check script pass | Restrictions: Do not rename AC IDs; mark anything not done as deferred with a reason; do not start spec-2 or spec-3 work | Success: All npm scripts green; every AC-LEAVE ID has a row with code and test links_
