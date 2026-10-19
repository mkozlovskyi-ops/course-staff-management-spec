> Spec 2 of 3 — leave decision. Depends on: spec-1-leave-request. Used by: spec-3-leave-balance.

# Tasks Document

Requirement references use this spec's `requirements.md` numbering (`R1`–`R4`, criteria as `R2.3`) plus the PM's AC
IDs and business rules (`BR-n`). Paths follow the developer project template
(`koldovsky/2026-quitcode-06-project-template`). Prerequisite: all spec-1 tasks are merged (tables, select RLS,
`private.is_manager_of`, `private.position_blocks`, seed and demo users, `messages.ts`, `my_records`, "My requests").

## 1. Shared types and messages

- [ ] 1.1. Add decision and record error codes to the message map
  - File: src/lib/leave/messages.ts
  - Add `DECIDE_COMMENT_REQUIRED`, `DECIDE_ALREADY_DECIDED`, `DECIDE_NOT_AWAITING`, `DECIDE_OWN_RECORD`,
    `DECIDE_FORBIDDEN`, `DECIDE_INVALID`, `RECORD_LOCKED`, `RECORD_FORBIDDEN` and a status-label map, with texts from
    design.md Component 4.
  - Purpose: one source for every user-facing text of this spec.
  - _Leverage: src/lib/leave/messages.ts (spec-1 messageFor), design.md Component 4_
  - _Requirements: R2.3, R2.5, R3.3, R4.2, R4.5 (AC-DECIDE-04, -07, -12, -17, -18)_
  - _Prompt: Role: TypeScript developer | Task: Extend spec-1's messageFor(code, params) with the 8 codes and the status-label map from design.md Component 4 so that RECORD_LOCKED with status approved returns exactly "This record is already approved and cannot be changed" and DECIDE_ALREADY_DECIDED with approved returns "Another manager has already decided: Approved" | Restrictions: Do not change or rename spec-1's existing codes or texts; texts must match the AC strings character for character; keep the ErrorCode union exhaustive | Success: TypeScript compiles; every new code yields its AC text; spec-1 message tests still pass_

- [ ] 1.2. Create pure record rules
  - File: src/lib/leave/record-rules.ts
  - `allowedOwnActions(status)`, `canBeDecided(status)`, `validateDecisionInput(decision, comment)`; `Decision` and
    `ActionResult` types.
  - Purpose: let the UI hide refused actions and pre-check the return comment, mirroring the SQL rules.
  - _Leverage: database.types.ts (status type from spec-1), src/lib/leave/messages.ts_
  - _Requirements: R2.3, R4.2, R4.5, BR-1, BR-5_
  - _Prompt: Role: TypeScript developer | Task: Implement the pure functions from design.md Component 5: edit allowed for awaiting/returned, resubmit only for returned, cancel for awaiting/returned, decide only for awaiting, return requires a non-blank trimmed comment | Restrictions: No I/O, no imports from Supabase or React; the database stays the authority | Success: Functions return the documented values for all 5 statuses and blank/whitespace comments_

- [ ] 1.3. Unit tests for messages and record rules
  - Files: tests/leave/messages.decide.test.ts, tests/leave/record-rules.test.ts
  - One `it()` per rule, titles with AC IDs, e.g. `it("AC-DECIDE-04: return without comment is refused")`.
  - Purpose: lock texts and status rules before the UI uses them.
  - _Leverage: vitest.config.mts, tests/leave/messages.test.ts (spec-1 style)_
  - _Requirements: R2.3, R2.5, R3.3, R4.2, R4.5 (AC-DECIDE-04, -07, -12, -17, -18)_
  - _Prompt: Role: QA engineer | Task: Write Vitest unit tests for tasks 1.1 and 1.2 covering every status and every new code, titled with the AC ID or BR number | Restrictions: No database or network; test both allowed and refused paths | Success: npm test passes; npm run ac:coverage counts these AC IDs_

## 2. Database functions and views

- [ ] 2.1. Implement `decide_record` with the approval hook
  - File: supabase/migrations/20261021090000_rpc_decide_record.sql
  - `private.on_approve(p_record public.leave_records, p_accept_split boolean)` no-op; `private.decide_record(p_record_id
    uuid, p_decision text, p_comment text default null, p_accept_split boolean default false)` following the 10 steps of
    design.md Component 1; `public.decide_record` security-invoker wrapper; `grant execute` to `authenticated` only.
  - Purpose: the single, race-safe write path for approve / reject / return, extensible by spec-3 with
    `create or replace`.
  - _Leverage: spec-1 tables leave_records and record_events, private.is_manager_of, private.current_app_role(), spec-1 public.submit_leave_request wrapper pattern_
  - _Requirements: R2 (AC-DECIDE-02, -03, -04, -05, -07, -10, -15), R3 (AC-DECIDE-08, -09, -18, -19), BR-2, BR-3, BR-4, BR-7_
  - _Prompt: Role: Postgres/Supabase developer | Task: Write the migration for decide_record exactly as design.md Component 1: auth check, select ... for update, admin-or-responsible-manager-not-own-record permission (DECIDE_OWN_RECORD vs DECIDE_FORBIDDEN), status check (DECIDE_ALREADY_DECIDED with detail = status, DECIDE_NOT_AWAITING for cancelled), mandatory trimmed comment for return, perform private.on_approve under the lock, update status/manager_comment/decided_by/decided_at (to the minute), insert one record_events row, return the row; raise errors as message = code, detail = parameter | Restrictions: security definer only in schema private with set search_path = '' and fully qualified names; auth.uid() only as (select auth.uid()); revoke execute from public and anon; do not write paid_days/unpaid_days or any balance logic (spec-3); keep the 4-argument signature so spec-3 can create or replace without changing it; timestamp newer than every spec-1 migration | Success: supabase db reset applies; calling rpc('decide_record') as each demo user behaves as AC-DECIDE-02..05, -07..-10, -18, -19 describe; a second concurrent call waits and then gets DECIDE_ALREADY_DECIDED_

- [ ] 2.2. Implement employee record functions
  - File: supabase/migrations/20261021090100_rpc_own_records.sql
  - `update_own_record(p_record_id, p_start, p_end)`, `resubmit_record(p_record_id)`, `cancel_own_record(p_record_id)`
    in `private` with `public` wrappers, as design.md Component 2.
  - Purpose: let the employee correct, resubmit or withdraw undecided records without touching decided ones.
  - _Leverage: spec-1 private.position_blocks, spec-1 validation codes (LEAVE_OUTSIDE_SCHEDULE, LEAVE_START_AT_BLOCK_END, LEAVE_END_BEFORE_START, LEAVE_OVERLAP), leave_records exclusion constraint_
  - _Requirements: R4 (AC-DECIDE-06, -12, -13, -16, -17), BR-1, BR-5, BR-6_
  - _Prompt: Role: Postgres/Supabase developer | Task: Implement the three functions: lock the record for update, require employee_id = (select auth.uid()) else RECORD_FORBIDDEN, enforce allowed statuses (update awaiting|returned keeping the status, resubmit returned → awaiting clearing manager_comment/decided_by/decided_at, cancel awaiting|returned → cancelled) else RECORD_LOCKED with detail = status; update_own_record re-validates hours against private.position_blocks and recomputes start_at, end_at and minutes like spec-1, mapping 23P01 to LEAVE_OVERLAP; write one record_events row per call (edited / resubmitted / cancelled) | Restrictions: Reuse spec-1's helper instead of copying schedule logic; do not allow changing date, position or leave type; managers and admins get RECORD_FORBIDDEN on other people's records | Success: Through the Data API AC-DECIDE-06, -12, -13, -16, -17 behave as written and an edit that overlaps another record is refused with no change_

- [ ] 2.3. Create `manager_queue` and `record_history` views
  - File: supabase/migrations/20261021090200_decision_views.sql
  - Both views `with (security_invoker = true)`; columns and filters from design.md Component 3; `grant select` to
    `authenticated`.
  - Purpose: role-filtered reads for the workspace tabs and the record history.
  - _Leverage: spec-1 select RLS on leave_records, positions, profiles, record_events; private.is_manager_of; private.current_app_role()_
  - _Requirements: R1 (AC-DECIDE-01, -11, -14), R2.7 (AC-DECIDE-15), R2.6 (AC-DECIDE-10), BR-3, BR-4, BR-7_
  - _Prompt: Role: Postgres/Supabase developer | Task: Create manager_queue (tab = awaiting for status awaiting, history otherwise; managers see records of positions they manage excluding their own records; admin sees all) and record_history (one row per record_events row with actor name and role label, time to the minute in Europe/Kyiv) | Restrictions: No security definer views; no new RLS policies on spec-1 tables; local times via the app_settings time zone from spec-1 | Success: For the seed data manager1 sees only main-position records, manager2 only additional-position ones, admin all, teacher1 none in manager_queue; record_history rows are visible to the employee, every responsible manager and admin only_

- [ ] 2.4. Regenerate database types
  - File: database.types.ts
  - Run `npm run db:types` after 2.1–2.3.
  - Purpose: typed `rpc('decide_record' | 'update_own_record' | 'resubmit_record' | 'cancel_own_record')` calls and
    view rows.
  - _Leverage: npm script db:types (template)_
  - _Requirements: R1, R2, R4_
  - _Prompt: Role: TypeScript developer | Task: Regenerate database.types.ts from the local database after applying this spec's migrations | Restrictions: Never hand-edit the generated file | Success: The new functions and views appear with correct argument and column types; npm run typecheck passes_

- [ ] 2.5. Database integration tests for decisions
  - File: tests/db/decide.test.ts
  - One `it()` per AC: AC-DECIDE-02, -03, -04, -05, -07, -08, -09, -10, -18, -19, plus BR-2 (cancelled record).
  - Purpose: prove the decision rules hold without the UI, for each role.
  - _Leverage: tests/db/helpers.ts (spec-1 sign-in and reset helpers), spec-1 seed and demo users, public.submit_leave_request to create fixture records_
  - _Requirements: R2, R3, BR-2_
  - _Prompt: Role: QA engineer | Task: Write Vitest integration tests that sign in as teacher1, manager1, manager2, a second manager of the main position and admin, create records via submit_leave_request, call decide_record and assert status, decided_by, manager_comment and the record_events row; for AC-DECIDE-07 run two clients, the first approves, the second rejects and must get DECIDE_ALREADY_DECIDED with detail approved | Restrictions: Reset data between tests; never use the service-role key for the actor under test; titles start with the AC ID | Success: All listed tests pass against supabase start_

- [ ] 2.6. Database integration tests for own records and views
  - Files: tests/db/own-records.test.ts, tests/db/decision-views.test.ts
  - Own records: AC-DECIDE-06, -12, -13, -16, -17, overlap refusal on edit, RECORD_FORBIDDEN. Views: AC-DECIDE-01,
    -11, -14, -15, BR-3, BR-4.
  - Purpose: prove the employee rules and the role filtering of both views.
  - _Leverage: tests/db/helpers.ts, spec-1 seed, tasks 2.2 and 2.3_
  - _Requirements: R1, R2.7, R4, BR-3, BR-4, BR-5_
  - _Prompt: Role: QA engineer | Task: Write Vitest integration tests for the employee functions and for manager_queue/record_history as each demo user, one it() per AC titled with its ID; for AC-DECIDE-15 assert the history line has decided_at to the minute, actor name and to_status approved and that teacher2 sees no rows | Restrictions: Reset data between tests; use anon-key clients signed in as demo users; do not test balances (spec-3) | Success: All listed tests pass; npm run ac:coverage finds tests for AC-DECIDE-01, -06, -11..-17_

## 3. Server actions

- [ ] 3.1. Implement `decideRecord`
  - File: src/app/(app)/records/[id]/actions.ts
  - `'use server'`; `decideRecord(recordId, decision, comment?)` → `ActionResult`; validates input shape, calls
    `rpc('decide_record', …)`, maps codes with `messageFor`, `revalidatePath('/manager')` and the card path.
  - Purpose: the UI's only way to decide; returns exact AC messages.
  - _Leverage: src/lib/supabase/server.ts createClient(), src/lib/leave/messages.ts, src/lib/leave/record-rules.ts_
  - _Requirements: R2, R3_
  - _Prompt: Role: Next.js 16 developer | Task: Implement the server action from design.md Component 6, reading error.message as the code and error.details as the parameter | Restrictions: No direct table writes; never decide on the client; unknown errors are logged server-side and return a generic message; leave room for an optional acceptSplit argument that spec-3 adds | Success: AC-DECIDE-04, -07 and -18 messages reach the UI exactly; a successful decision revalidates the queue and the card_

- [ ] 3.2. Implement own-record actions
  - File: src/app/(app)/requests/record-actions.ts
  - `updateOwnRecord(recordId, { start, end } | { wholeDay: true })`, `resubmitRecord(recordId)`,
    `cancelOwnRecord(recordId)` → `ActionResult`; revalidate `/requests` and `/manager`.
  - Purpose: employee edits, resubmits and cancellations from "My requests".
  - _Leverage: src/lib/supabase/server.ts, src/lib/leave/messages.ts, src/lib/leave/schedule.ts (spec-1)_
  - _Requirements: R4_
  - _Prompt: Role: Next.js 16 developer | Task: Implement the three server actions calling the RPCs from task 2.2; for wholeDay resolve the record's first block start and last block end server-side | Restrictions: Separate file from spec-1's submitRequest action; validate HH:MM on a 5-minute grid before calling the RPC; no direct table writes | Success: AC-DECIDE-12 message shown on an approved record; edit, resubmit and cancel update the lists after revalidation_

## 4. Frontend

- [ ] 4.1. Build the manager workspace
  - Files: src/app/(app)/manager/page.tsx, src/app/(app)/manager/QueueTable.tsx
  - Tabs "Awaiting decision" (default) and "History" via `?tab=&page=`, server-side pagination of 50, empty state
    "No records awaiting decision", rows linking to `/records/[id]`; navigation link for manager and admin roles.
  - Purpose: the manager's list of records to decide and of records already handled.
  - _Leverage: view manager_queue, src/lib/supabase/server.ts, spec-1 app layout and navigation_
  - _Requirements: R1 (AC-DECIDE-01, -11, -14)_
  - _Prompt: Role: React/Next.js developer | Task: Build the server-rendered workspace from design.md Component 7 reading manager_queue filtered by tab | Restrictions: No client-side filtering of other managers' data (RLS and the view do it); times shown in Europe/Kyiv; desktop-first | Success: manager1 sees only main-position records; after an approval the record moves from the first tab to History; a manager with none sees the exact empty-state text_

- [ ] 4.2. Build the record card with history
  - Files: src/app/(app)/records/[id]/page.tsx, src/app/(app)/records/[id]/RecordHistory.tsx
  - Details (employee, position, date, hours, leave type, status, manager comment) and history
    lines "date time — name (role) — action — comment"; `notFound()` when RLS hides the record.
  - Purpose: one card for the employee, managers and admin to see the record and who decided what and when.
  - _Leverage: views manager_queue / my_records (spec-1), record_history_
  - _Requirements: R2.6, R2.7 (AC-DECIDE-10, -15), R2.4 (AC-DECIDE-05)_
  - _Prompt: Role: React/Next.js developer | Task: Build the server component card and the history list from design.md Component 8 | Restrictions: Read only through views; show times to the minute in Europe/Kyiv; do not render the decision panel here (task 4.3) | Success: After an approval the card shows a line with date, time, decider and "Approved"; an admin decision shows the administrator as decider_

- [ ] 4.3. Build the decision panel
  - File: src/app/(app)/records/[id]/DecisionPanel.tsx
  - Client component: Approve, Reject, Return for revision with comment textarea; pre-check with
    `validateDecisionInput`; shows server messages; rendered by the card only when the viewer may decide and status is
    "Awaiting decision".
  - Purpose: the manager's decision controls.
  - _Leverage: src/lib/leave/record-rules.ts, decideRecord (task 3.1), src/app/(app)/records/[id]/page.tsx_
  - _Requirements: R2 (AC-DECIDE-02..05, -07), R3 (AC-DECIDE-18)_
  - _Prompt: Role: React developer | Task: Implement the panel calling decideRecord with useActionState/useTransition, disabling buttons while pending and showing the returned message | Restrictions: Server is the authority — show refusals, never assume success; leave a slot for spec-3's balance warning and split choice | Success: Return without comment shows "Tell the employee what to fix"; a race shows "Another manager has already decided: Approved" and the card refreshes_

- [ ] 4.4. Add own-record controls to "My requests"
  - Files: src/app/(app)/requests/OwnRecordControls.tsx, src/app/(app)/requests/EditRecordDialog.tsx,
    src/app/(app)/requests/RecordRow.tsx
  - Edit hours (whole day / hours on a 5-minute grid, instant validation), Save and resubmit, Cancel with
    confirmation, History link; manager comment under returned / rejected rows; one-line change in spec-1's
    `RecordRow.tsx` to render the controls.
  - Purpose: let employees act on their records and see decisions.
  - _Leverage: src/lib/leave/record-rules.ts allowedOwnActions, src/lib/leave/schedule.ts validateInterval (spec-1), task 3.2 actions, spec-1 RecordRow.tsx and my_records_
  - _Requirements: R2.2, R2.4 (AC-DECIDE-03, -05), R4 (AC-DECIDE-06, -12, -13, -16, -17)_
  - _Prompt: Role: React/Next.js developer | Task: Build the controls and edit dialog from design.md Component 9 and render them from RecordRow | Restrictions: Change nothing else in spec-1's page; hide actions not allowed by status but still show server refusals; times in Europe/Kyiv | Success: A returned record can be edited and resubmitted; an approved record shows no edit control and a direct attempt shows the AC-DECIDE-12 message; a cancelled record offers no resubmit_

## 5. Integration, E2E and wrap-up

- [ ] 5.1. End-to-end decision flow tests
  - File: tests/e2e/decision-flow.spec.ts
  - Return without / with comment, employee edits and resubmits, approve → History and history line, empty state.
  - Purpose: verify the UI-visible behaviour across roles in a real browser.
  - _Leverage: Playwright setup from spec-1 (playwright.config.ts), demo users and seed_
  - _Requirements: R1 (AC-DECIDE-11, -14), R2 (AC-DECIDE-04, -05, -15), R4 (AC-DECIDE-06)_
  - _Prompt: Role: QA automation engineer | Task: Write Playwright tests titled with AC IDs that sign in as manager1 and teacher1 and walk the flow from design.md "End-to-End Testing" | Restrictions: Only if the course allows Playwright (spec-1 decides); otherwise record these ACs as manually verified in docs/conformance.md with steps; reset the database before the suite | Success: Tests pass locally against npm run dev with a reset database_

- [ ] 5.2. Conformance rows and cleanup
  - Files: docs/conformance.md, docs/spec-reading.md
  - One row per AC-DECIDE-01..19 (19 rows) with status, code link and test link; note the `p_accept_split` /
    `private.on_approve` extension point for spec-3; remove dead code; run `npm run lint`, `typecheck`, `test`,
    `ac:coverage`, `spec:check`, `scripts/check-migration-order.mjs`.
  - Purpose: close the spec with traceable evidence for every AC.
  - _Leverage: scripts/check-ac-coverage.mjs, scripts/check-spec.mjs, scripts/check-migration-order.mjs_
  - _Requirements: All_
  - _Prompt: Role: Senior developer closing a spec | Task: Fill docs/conformance.md for all 19 AC-DECIDE IDs and make every check script pass | Restrictions: Do not rename AC IDs; do not touch spec-1 rows except to link shared files; mark anything not done as deferred with a reason | Success: All npm scripts green; every AC-DECIDE ID has a row with code and test links_
