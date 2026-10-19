Spec 3 of 3 — leave balance. Depends on: spec-1-leave-request, spec-2-leave-decision. Used by: none.

# Tasks Document

Requirement references use this spec's numbering `<requirement>.<criterion>` (for example `2.3` = Requirement 2,
criterion 3) plus the PM's AC ID. Mapping: 1.1–1.8 = AC-BALANCE-01..07, -11; 2.1–2.6 = AC-BALANCE-08, -15, -09, -14,
-10, -16; 3.1–3.2 = AC-BALANCE-12, -13. Everything marked "from spec-1" / "from spec-2" must already be merged before
starting; this spec does not redefine it (spec-1 migrations end at `20261020090400`, spec-2's are
`20261021090000`–`20261021090200`). Migration timestamps start at `20261022090000`. spec-2's `decide_record` signature
is never changed: the balance step goes into its hook `private.on_approve`.

## 1. Database schema

- [ ] 1.1. Add annual limit, carry-over table, snapshot columns and the visibility helper
  - File: supabase/migrations/20261022090000_balance_schema.sql
  - Column `leave_types.annual_limit` (sick/personal 18.00, unpaid/family null); table `balance_carryovers` with RLS (select only, via `private.can_see_employee`); columns `paid_days`,
    `unpaid_days` on `leave_records` with the two check constraints from design.md "Data Models";
    `private.can_see_employee(p_employee uuid)` (security definer, stable) with execute granted to `authenticated`.
  - Purpose: storage for granted days, carry-over and the per-record paid/unpaid snapshot; one place that decides
    who may see an employee's balances.
  - _Leverage: supabase/migrations/20261012000000_profiles_and_roles.sql (RLS conventions), spec-1 leave tables
    (20261020090200_leave_records.sql), private.is_manager_of, private.is_admin_
  - _Requirements: 1.1 (AC-BALANCE-01), 1.8 (AC-BALANCE-11), 2.5 (AC-BALANCE-10), 3.1 (AC-BALANCE-12), 3.2
    (AC-BALANCE-13)_
  - _Prompt: Role: Postgres/Supabase developer | Task: Write the migration adding public.leave_types.annual_limit numeric(5,2) null (check >= 0) and setting it to 18.00 for sick and personal, null for unpaid and family; creating public.balance_carryovers (PK employee_id, leave_type, school_year; leave_type limited to sick/personal; days >= 0), adding nullable paid_days and unpaid_days numeric(5,2) to public.leave_records with the checks from design.md, and private.can_see_employee returning true for self, private.is_admin(), or a responsible manager (private.is_manager_of) of any of the employee's positions | Restrictions: Do not edit spec-1 or spec-2 migrations; do not change leave_types' existing columns or rows other than setting annual_limit; RLS enabled, revoke all from anon and authenticated, then grant select only; auth.uid() only as (select auth.uid()); security definer only in schema private with set search_path = ''; if spec-1 uses column-level select grants on leave_records, grant the new columns the same way | Success: supabase db reset applies cleanly; leave_types shows 18.00/18.00/null/null; teacher1 can select own carry-over rows only; manager1 sees rows of employees of managed positions only; admin sees all; inserting paid_days without unpaid_days fails_

- [ ] 1.2. Create balance functions
  - File: supabase/migrations/20261022090100_balance_functions.sql
  - `private.day_share`, `private.school_year_of`, `private.first_school_year`, `private.record_days`,
    `private.balance_for`, `private.request_balance_warning`, `private.record_balance_preview` and the public wrapper
    `public.record_balance_preview(record_id uuid)`; backfill `paid_days`/`unpaid_days` for already approved dev rows.
  - Purpose: the authoritative arithmetic for used, carried-over and remaining days, warnings and previews.
  - _Leverage: design.md Components 4 and "Day-share rule", leave_types.annual_limit, leave_records.minutes and
    day_total_minutes from spec-1_
  - _Requirements: 1.1–1.8 (AC-BALANCE-01..07, -11), 2.1–2.4 (AC-BALANCE-08, -15, -09, -14), 3.1, 3.2_
  - _Prompt: Role: Postgres developer | Task: Implement the functions exactly as specified in design.md Component 4: day_share = least(1.00, round(minutes::numeric / total, 2)); school_year_of = extract(year from d - interval '8 months'); record_days as the incremental share over the employee's other approved minutes on the date (all positions, all types); balance_for returning granted, carried_over (explicit carry-over row, else previous year's remaining, from first_school_year), used (paid_days per type, unpaid_days for Unpaid leave) and remaining, and nothing unless can_see_employee; request_balance_warning returning null or {leaveType, leaveTypeName, items[]} with one item per exceeded school year; record_balance_preview returning null for invisible, non-awaiting or unlimited records | Restrictions: Never hard-code 18.00 — read leave_types.annual_limit; only status 'approved' counts; numeric arithmetic only, no float; security definer only in private with search_path = ''; leave type read via leave_records.request_id → leave_requests.leave_type | Success: For seeded data the functions return 0.36 for 150/420, 0.15 for 75/495, 1.00 for both AC-BALANCE-05 records, and 2027-2028 carried_over 5.00 when 2026-2027 remaining is 5.00; an invisible employee yields zero rows_

- [ ] 1.3. Create the leave_balances view
  - File: supabase/migrations/20261022090200_leave_balances_view.sql
  - `public.leave_balances` with `security_invoker = true`: distinct `employee_id` from `positions` × school years
    from `private.first_school_year` to the current Kyiv school year + 1 × lateral `private.balance_for`; grant select
    to `authenticated`.
  - Purpose: one role-filtered read for the Balances pages.
  - _Leverage: spec-1 positions RLS, private.balance_for from 1.2_
  - _Requirements: 1.1–1.8 (AC-BALANCE-01..07, -11), 3.1 (AC-BALANCE-12), 3.2 (AC-BALANCE-13)_
  - _Prompt: Role: Postgres developer | Task: Create the view described in design.md Component 5 with columns employee_id, school_year, leave_type, leave_type_name, granted, carried_over, used, remaining | Restrictions: security_invoker = true, never a security definer view; use (now() at time zone 'Europe/Kyiv')::date for the current school year; no money or salary columns | Success: teacher1 sees only own rows; manager1 sees rows only for employees of managed positions; admin sees all; AC-BALANCE-01 values appear for a fresh employee_

- [ ] 1.4. Seed demo carry-over
  - File: scripts/seed-leave-demo.mjs
  - Append `balance_carryovers` rows for school year 2026 (0.00 for teacher1, a non-zero demo value for teacher2) so the
    Balances page shows carry-over locally.
  - Purpose: make AC-BALANCE-01 and the carried-over column reproducible by hand.
  - _Leverage: spec-1 scripts/seed-leave-demo.mjs (resolves demo users by email, service-role, idempotent),
    supabase/demo-users.json_
  - _Requirements: 1.1 (AC-BALANCE-01), 1.8 (AC-BALANCE-11)_
  - _Prompt: Role: Developer preparing test fixtures | Task: Extend spec-1's seed-leave-demo.mjs with an idempotent upsert of balance_carryovers rows for demo users resolved by email | Restrictions: Do not change spec-1's existing seed steps; supabase/seed.sql stays user-independent; only @example.test users; synthetic values only; local use only | Success: supabase db reset, npm run users:demo and node scripts/seed-leave-demo.mjs show teacher1 with granted 18.00, carried over 0.00, remaining 18.00 for Sick and Personal leave_

## 2. Replaced RPCs

- [ ] 2.1. Return the over-balance warning from submit_leave_request
  - File: supabase/migrations/20261022090300_submit_balance_warning.sql
  - `create or replace` spec-1's `private.submit_leave_request(payload jsonb) returns jsonb` with the same signature,
    return type and grants, adding `warning := private.request_balance_warning(request_id)` to the result
    `{request_id, record_ids, warning}`.
  - Purpose: the employee learns at submission that part of the request will be Unpaid leave.
  - _Leverage: spec-1 submit_leave_request body (20261020090300_rpc_submit.sql), private.request_balance_warning
    from 1.2_
  - _Requirements: 2.1 (AC-BALANCE-08), 2.2 (AC-BALANCE-15)_
  - _Prompt: Role: Postgres developer | Task: Copy spec-1's submit_leave_request body unchanged and add the warning computation after the records are inserted, returning spec-1's result plus the key warning (null when the balance is not exceeded) | Restrictions: The warning never blocks or rolls back the request; do not change the signature, validation, error codes or grants from spec-1; do not touch the public wrapper | Success: With Personal leave remaining 0.50 a whole-day request returns a warning item {remaining 0.50, paid 0.50, unpaid 0.50}; with remaining 0.20 a 09:00–11:30 request on a 420-minute day returns {0.20, 0.20, 0.16}; otherwise warning is null; spec-1's integration tests still pass_

- [ ] 2.2. Replace spec-2's approval hook with the balance check and split
  - File: supabase/migrations/20261022090400_on_approve_balance.sql
  - `create or replace function private.on_approve(p_record public.leave_records, p_accept_split boolean) returns void`:
    per-employee advisory lock, `record_days`, remaining for the record's school year, `BALANCE_SPLIT_REQUIRED` when
    exceeded without `p_accept_split`, otherwise update `paid_days` / `unpaid_days`. `decide_record` is not touched.
  - Purpose: paid leave is never overspent, even under concurrent approvals or direct API calls.
  - _Leverage: spec-2 private.on_approve no-op and decide_record (20261021090000_rpc_decide_record.sql, which calls the
    hook for approve under the row lock before setting the status), private.record_days and private.balance_for
    from 1.2_
  - _Requirements: 1.2 (AC-BALANCE-02), 1.3 (AC-BALANCE-03), 1.5 (AC-BALANCE-05), 1.6 (AC-BALANCE-06), 1.7
    (AC-BALANCE-07), 2.4 (AC-BALANCE-14), 2.5 (AC-BALANCE-10), 2.6 (AC-BALANCE-16)_
  - _Prompt: Role: Postgres developer | Task: Replace private.on_approve per design.md Component 7 with the exact signature spec-2 declared, raising errors as raise exception using errcode = 'P0001', message = 'BALANCE_SPLIT_REQUIRED', detail = json {leaveType, remaining} | Restrictions: Do not change decide_record, its signature or grants; security definer with set search_path = ''; never trust balance numbers from the client; Unpaid leave records store paid_days 0 and unpaid_days = days; unlimited types are never refused; reject and return never reach the hook | Success: Approving AC-BALANCE-09's record with p_accept_split false raises BALANCE_SPLIT_REQUIRED and leaves it awaiting; with true the returned row has paid_days 0.50 and unpaid_days 0.50; spec-2's integration tests still pass unchanged_

## 3. Domain (pure functions)

- [ ] 3.1. Day arithmetic
  - File: src/lib/leave/duration.ts
  - `dayShareHundredths`, `dayShare`, `incrementalDayShare`, `formatDays`.
  - Purpose: the TS mirror of `private.day_share` used for display and parity tests.
  - _Leverage: design.md Component 1_
  - _Requirements: 1.3 (AC-BALANCE-03), 1.5 (AC-BALANCE-05), 1.6 (AC-BALANCE-06)_
  - _Prompt: Role: TypeScript developer | Task: Implement the four functions with integer hundredths: min(100, floor((200 * minutes + total) / (2 * total))) | Restrictions: Pure, no imports, no floating-point equality, throw on total <= 0 | Success: dayShare(150, 420) = 0.36, dayShare(75, 495) = 0.15, dayShare(71, 200) = 0.36, dayShare(600, 420) = 1, incrementalDayShare(420, 75, 495) = 0.15, formatDays(0.5) = "0.50"_

- [ ] 3.2. School year and split
  - File: src/lib/leave/balance.ts
  - `schoolYearOf`, `schoolYearLabel`, `suggestSplit`.
  - Purpose: default year of the Balances page and the TS mirror of the split rule.
  - _Leverage: src/lib/leave/duration.ts (hundredths helpers)_
  - _Requirements: 1.8 (AC-BALANCE-11), 2.1 (AC-BALANCE-08), 2.2 (AC-BALANCE-15)_
  - _Prompt: Role: TypeScript developer | Task: Implement the functions from design.md Component 2 working on ISO date strings and hundredths | Restrictions: No Date time-zone conversions inside — the caller passes the Kyiv date as "YYYY-MM-DD"; pure functions only | Success: schoolYearOf("2027-08-31") = 2026, schoolYearOf("2027-09-01") = 2027, schoolYearLabel(2027) = "2027–2028", suggestSplit(1, 0.5) = {paid 0.5, unpaid 0.5, exceeded true}, suggestSplit(0.36, 0.2) = {paid 0.2, unpaid 0.16, exceeded true}, suggestSplit(1, 1.5) = {paid 1, unpaid 0, exceeded false}_

- [ ] 3.3. Balance message codes
  - File: src/lib/leave/messages.ts
  - Add `BALANCE_WARNING_SUBMIT`, `BALANCE_WARNING_DECIDE`, `BALANCE_SPLIT_REQUIRED`.
  - Purpose: exact AC texts from one map.
  - _Leverage: spec-1 messageFor map and parseRpcError (params from JSON in error.details), formatDays from 3.1_
  - _Requirements: 2.1 (AC-BALANCE-08), 2.2 (AC-BALANCE-15), 2.3 (AC-BALANCE-09), 2.4 (AC-BALANCE-14), 2.6
    (AC-BALANCE-16)_
  - _Prompt: Role: TypeScript developer | Task: Extend messageFor with the three texts quoted in design.md Component 3, formatting numbers with formatDays and using the leave type's display name | Restrictions: Texts must match the AC strings character for character; do not change existing spec-1/spec-2 entries | Success: messageFor("BALANCE_SPLIT_REQUIRED", {leaveType: "Personal leave", remaining: 0.5}) returns "The Personal leave balance is 0.50 day: approve with the suggested split or reject"_

- [ ] 3.4. Unit tests for balance arithmetic and messages
  - Files: tests/leave/duration.test.ts, tests/leave/balance.test.ts, tests/leave/messages-balance.test.ts
  - One `it()` per AC, titled "AC-BALANCE-NN: …", plus rounding and cap edge cases.
  - Purpose: lock the arithmetic and texts; catch regressions.
  - _Leverage: vitest.config.mts, src/lib/leave/duration.ts, balance.ts, messages.ts_
  - _Requirements: 1.3, 1.5, 1.6, 1.8, 2.1, 2.2, 2.3, 2.4, 2.6 (AC-BALANCE-03, -05, -06, -11, -08, -15, -09, -14, -16)_
  - _Prompt: Role: QA engineer | Task: Write Vitest unit tests for the functions and messages above as listed in design.md "Unit Testing" | Restrictions: No database or network; test success and edge paths (0.355 → 0.36, cap 1.00, remaining 0, not exceeded) | Success: npm test passes and npm run ac:coverage finds a test for each listed AC_

## 4. Server actions

- [ ] 4.1. Pass balance warnings from submitRequest
  - File: src/app/(app)/requests/actions.ts
  - `submitRequest` returns `{ ok: true, requestId, warnings: string[] }`, with each item of the RPC's `warning`
    turned into text via `messageFor("BALANCE_WARNING_SUBMIT", …)`.
  - Purpose: the form can show the warning after the request is created.
  - _Leverage: spec-1 submitRequest, src/lib/leave/messages.ts, database.types.ts (regenerate with npm run db:types)_
  - _Requirements: 2.1 (AC-BALANCE-08), 2.2 (AC-BALANCE-15)_
  - _Prompt: Role: Next.js developer | Task: Extend spec-1's submitRequest to read warning from the RPC result and return translated warnings | Restrictions: Do not change spec-1's error handling or validation; warnings never turn a success into a failure; regenerate types instead of hand-writing them | Success: The action returns one AC-BALANCE-08 warning string for the AC scenario and an empty array otherwise; spec-1's tests still pass_

- [ ] 4.2. Add acceptSplit to decideRecord
  - File: src/app/(app)/records/[id]/actions.ts
  - `decideRecord(recordId, decision, comment?, acceptSplit = false)` passes `p_accept_split` (the argument spec-2
    reserved); maps
    `BALANCE_SPLIT_REQUIRED` via `messageFor` and revalidates the card and `/balances` paths.
  - Purpose: the manager can approve with the split; refusals show the exact text.
  - _Leverage: spec-2 decideRecord in src/app/(app)/records/[id]/actions.ts, src/lib/leave/messages.ts_
  - _Requirements: 2.5 (AC-BALANCE-10), 2.6 (AC-BALANCE-16)_
  - _Prompt: Role: Next.js developer | Task: Extend spec-2's decideRecord with the acceptSplit parameter and the new error code | Restrictions: Never compute the split in the action; keep spec-2's existing messages and refresh behaviour | Success: Approving without split returns the AC-BALANCE-16 message; approving with split returns ok and the card shows Approved_

## 5. Frontend

- [ ] 5.1. Balances pages and table
  - Files: src/app/(app)/balances/page.tsx, src/app/(app)/balances/[employeeId]/page.tsx,
    src/app/(app)/balances/BalanceTable.tsx
  - Own balances and another employee's balances from `leave_balances`; year selection via `?year=`; four columns for
    limited types, "used" only for unlimited; neutral "Balances are not available" state for zero rows; "Balances" link
    in spec-1's app navigation.
  - Purpose: employees, managers and the administrator see balances per the permission matrix.
  - _Leverage: src/lib/supabase/server.ts, src/lib/leave/balance.ts, src/lib/leave/duration.ts, spec-1 app layout_
  - _Requirements: 1.1–1.8 (AC-BALANCE-01..07, -11), 3.1 (AC-BALANCE-12), 3.2 (AC-BALANCE-13)_
  - _Prompt: Role: React/Next.js developer | Task: Build server-rendered pages reading leave_balances for the employee and school year (default schoolYearOf(today in Europe/Kyiv)) and a presentational BalanceTable formatting every number with formatDays and the year with schoolYearLabel | Restrictions: No client-side balance computation; the same empty state for forbidden and unknown employees; only a numeric year in the URL; desktop-first | Success: AC-BALANCE-01 and -11 render exactly as written; manager1 opening teacher1's page in the AC-BALANCE-12 setup and teacher1 opening another employee's page both see "Balances are not available"_

- [ ] 5.2. Balance split panel on the record card
  - Files: src/app/(app)/records/[id]/BalanceSplitPanel.tsx, src/app/(app)/records/[id]/page.tsx,
    src/app/(app)/records/[id]/DecisionPanel.tsx
  - Card page loads `record_balance_preview`; panel shows `BALANCE_WARNING_DECIDE` and a "View balances" link;
    DecisionPanel's Approve becomes "Approve with split" (acceptSplit true) when exceeded.
  - Purpose: the manager sees the warning and suggested split before deciding.
  - _Leverage: spec-2 card page and DecisionPanel, decideRecord from 4.2, messageFor from 3.3_
  - _Requirements: 2.3 (AC-BALANCE-09), 2.4 (AC-BALANCE-14), 2.5 (AC-BALANCE-10), 2.6 (AC-BALANCE-16)_
  - _Prompt: Role: React/Next.js developer | Task: Add the panel and wire it into spec-2's card as in design.md Component 7 | Restrictions: The preview is loaded on every page render (no caching) so it reflects approvals made in between; keep spec-2's reject and return flows unchanged; show server messages verbatim | Success: For the AC-BALANCE-09 record the card shows the exact warning; after approving the first AC-BALANCE-14 record, opening the second shows the 0.50/0.50 warning_

- [ ] 5.3. Balance warning in the New request form
  - Files: src/app/(app)/requests/new/BalanceWarning.tsx, src/app/(app)/requests/new/RequestForm.tsx
  - After a successful submit with warnings, the form shows a success state with each warning and a link to "My
    requests"; without warnings spec-1's redirect is unchanged.
  - Purpose: the employee is told the request was created and how much becomes Unpaid leave.
  - _Leverage: spec-1 RequestForm, submitRequest from 4.1_
  - _Requirements: 2.1 (AC-BALANCE-08), 2.2 (AC-BALANCE-15)_
  - _Prompt: Role: React/Next.js developer | Task: Add a presentational BalanceWarning component and show it from RequestForm's success branch | Restrictions: The warning never blocks or undoes submission; do not change spec-1's validation or summary dialog | Success: The AC-BALANCE-08 and -15 texts are visible after submit and the request appears in "My requests"_

## 6. Integration, E2E and wrap-up

- [ ] 6.1. Database integration tests for balances and visibility
  - Files: tests/db/balances.test.ts, tests/db/balance-rls.test.ts, tests/db/balance-fixtures.ts
  - Fixtures (service role, setup only) create approved history and carry-over; tests sign in as demo users.
  - Purpose: prove balance values and privacy without the UI.
  - _Leverage: tests/db/helpers.ts (spec-1), demo users and seed_
  - _Requirements: 1.1–1.8 (AC-BALANCE-01..07, -11), 3.1 (AC-BALANCE-12), 3.2 (AC-BALANCE-13)_
  - _Prompt: Role: QA engineer | Task: Write Vitest integration tests, one it() per AC titled with its ID, querying leave_balances, balance_carryovers and record_balance_preview as the acting demo user | Restrictions: Reset data between tests; never use the service-role key for the actor under test; AC-BALANCE-11 fixture leaves Sick leave remaining 5.00 in 2026 and reads the 2027 row | Success: All listed ACs pass, including AC-BALANCE-04 (an awaiting record leaves used 0.00) and AC-BALANCE-07 (a rejected record leaves used 0.00); manager1 and teacher1 get zero rows in the AC-BALANCE-12 and -13 setups_

- [ ] 6.2. Database integration tests for warnings, split and concurrency
  - Files: tests/db/balance-split.test.ts, tests/db/balance-parity.test.ts
  - Submit warnings, preview, approve with and without split, two parallel approvals; SQL vs TS parity matrix.
  - Purpose: prove the split rule holds through the Data API and that SQL and TS agree.
  - _Leverage: tests/db/helpers.ts, tests/db/balance-fixtures.ts from 6.1, src/lib/leave/duration.ts, balance.ts_
  - _Requirements: 2.1–2.6 (AC-BALANCE-08, -15, -09, -14, -10, -16), 1.3, 1.6_
  - _Prompt: Role: QA engineer | Task: Write integration tests for submit_leave_request (task 2.1) and decide_record with p_accept_split going through the hook from task 2.2 and a parity test calling private functions through a test-only service-role connection | Restrictions: The concurrency test fires two decide_record calls for different records of one employee in parallel and asserts total paid_days never exceeds the balance; titles carry AC IDs | Success: All six ACs pass; parallel approvals never overspend; parity matrix has no mismatches_

- [ ] 6.3. End-to-end balance flow
  - File: tests/e2e/balance-flow.spec.ts
  - Submit over balance → warning → manager card warning → approve without split refused → approve with split →
    Balances page values.
  - Purpose: verify the user-visible flow across the three specs.
  - _Leverage: spec-1 playwright.config.ts, demo users, tests/db/balance-fixtures.ts_
  - _Requirements: 2.1 (AC-BALANCE-08), 2.3 (AC-BALANCE-09), 2.5 (AC-BALANCE-10), 2.6 (AC-BALANCE-16), 1.1
    (AC-BALANCE-01)_
  - _Prompt: Role: QA automation engineer | Task: Add one Playwright test per step titled with AC IDs | Restrictions: Use spec-1's Playwright config, do not add another runner; if Playwright was not allowed in the course (spec-1's fallback), record these steps as manually verified in docs/conformance.md | Success: The flow passes locally against npm run dev with a reset database, or the manual verification rows exist_

- [ ] 6.4. Conformance rows and cleanup
  - Files: docs/conformance.md, database.types.ts
  - One row per AC-BALANCE ID (16) with status, code link and test link; regenerate types; remove dead code and
    leftover debug output; run `npm run lint`, `typecheck`, `test`, `ac:coverage`, `spec:check`.
  - Purpose: close the spec with every AC traceable and all checks green.
  - _Leverage: scripts/check-ac-coverage.mjs, scripts/check-spec.mjs, scripts/check-migration-order.mjs_
  - _Requirements: All_
  - _Prompt: Role: Senior developer closing a slice | Task: Fill docs/conformance.md for AC-BALANCE-01..16, regenerate database.types.ts and make every check script pass | Restrictions: Do not rename AC IDs or edit spec-1/spec-2 rows; mark anything not done as deferred with a reason | Success: All npm scripts green; 16 AC-BALANCE rows with code and test links; spec-1 and spec-2 test suites still pass_
