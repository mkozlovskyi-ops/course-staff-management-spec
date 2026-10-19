> Spec 2 of 3 — leave decision. Depends on: spec-1-leave-request. Used by: spec-3-leave-balance.

# Requirements Document

> Source: `docs/requirements/02-leave-decision.md` and `docs/product.md` (PM repo, 2026-10-19), split from the combined
> spec `.claude/specs/leave-management-ws6/requirements.md` (its Requirements 5–8). Every acceptance criterion keeps the
> PM's AC ID so tests and conformance tables can be named after it. If this document and the PM text disagree, the PM
> text wins.

## Introduction

Part of slice WS6 of the school staff leave portal. Spec-1 (`spec-1-leave-request`) lets an employee submit a leave
request that becomes one leave record per position per date, with status "Awaiting decision", and shows the
employee's own records read-only in "My requests". This spec adds the decision step: each responsible manager sees
only the records of the positions they are assigned to, decides on each record separately (approve, reject, or return
for revision with a comment), and the employee sees the result and the comment. The employee can correct or cancel a
record that is not yet decided and resubmit a returned record. An administrator can decide any record, including
records of positions with no responsible manager and managers' own records.

Leave balances, the over-balance warning and the Personal/Unpaid split at approval time belong to
`spec-3-leave-balance` (see spec-3); this spec only leaves room for them.

## Alignment with Product Vision

- Success indicator "Each manager sees only the records of their own positions in the 'Awaiting decision' list and
  decides on each record separately" → Requirements 1–3.
- Permission matrix in `docs/product.md`: "Approve, reject, return for revision or cancel a record — manager: only
  records of own positions, never own records; admin: yes" → Requirement 3; "Change own record with status 'Awaiting
  decision' or 'Returned for revision' — own only" → Requirement 4.
- Removes the "who approves?" ambiguity for employees with two positions: each position's record goes to that
  position's managers only.
- Out of scope (PM "Out of scope"): a manager editing a record before deciding and cancelling approved records (WS11),
  deputy managers (WS10), the balance warning (spec-3), the school-calendar conflict warning (WS7), emails, and
  approving all records of a request in one action.

## Requirements

### Requirement 1 — Manager "Awaiting decision" list and history

**User Story:** As a responsible manager, I want a list of records awaiting my decision for my positions only, and a
history of records that left it, so that I can process them without seeing other divisions' data.

#### Acceptance Criteria

1. **AC-DECIDE-01** WHEN an employee with a main position (manager manager1@example.test) and an additional position
   (manager manager2@example.test) has requested a whole day with both positions AND manager1@example.test opens the
   "Awaiting decision" list THEN the portal SHALL list this employee's main-position record AND SHALL NOT list the
   additional-position record.
2. **AC-DECIDE-11** WHEN the manager's "Awaiting decision" list has 3 records AND the manager approves 1 of them THEN 2
   records SHALL remain in "Awaiting decision" AND the approved record SHALL appear in the manager's "History".
3. **AC-DECIDE-14** WHEN the manager has 0 records with status "Awaiting decision" AND opens the "Awaiting decision"
   list THEN the portal SHALL show "No records awaiting decision".

### Requirement 2 — Decide on a record

**User Story:** As a responsible manager, I want to approve, reject or return each record separately, with a comment
when I return it, so that one decision never changes the employee's other records and the employee knows what to fix.

#### Acceptance Criteria

1. **AC-DECIDE-02** WHEN a request has 2 records with status "Awaiting decision" AND the manager approves one of them
   THEN that record SHALL have status "Approved" AND the other SHALL stay "Awaiting decision".
2. **AC-DECIDE-03** WHEN the manager rejects a record with status "Awaiting decision" THEN the record SHALL have status
   "Rejected" AND the employee SHALL see this status in "My requests".
3. **AC-DECIDE-04** IF the manager returns a record with status "Awaiting decision" for revision without a comment THEN
   the status SHALL NOT change AND the portal SHALL show the manager "Tell the employee what to fix".
4. **AC-DECIDE-05** WHEN the manager returns a record with status "Awaiting decision" for revision with the comment
   "Enter hours instead of a whole day" THEN the record SHALL have status "Returned for revision" AND the employee SHALL
   see the comment "Enter hours instead of a whole day" on it.
5. **AC-DECIDE-07** WHEN a position has 2 responsible managers who both see its record with status "Awaiting decision"
   AND the first manager approves the record AND the second then tries to reject it THEN the record SHALL stay
   "Approved" AND the second manager SHALL see "Another manager has already decided: Approved".
6. **AC-DECIDE-10** WHEN the administrator approves a record with status "Awaiting decision" of a position for which
   the administrator is not a responsible manager THEN the record SHALL have status "Approved" AND the record's history
   SHALL show that the administrator made the decision.
7. **AC-DECIDE-15** WHEN a manager has approved a record AND the employee or any responsible manager of the position
   views the record's history THEN the history SHALL contain a line with the date and time of the decision, who
   decided, and the decision "Approved".

### Requirement 3 — Decision access control

**User Story:** As the school, I want only the right people to decide a record, so that nobody approves their own leave
or leave of a position they are not responsible for.

#### Acceptance Criteria

1. **AC-DECIDE-08** IF manager2@example.test is not a responsible manager of the employee's main position AND tries to
   approve a record of that position THEN the status SHALL NOT change AND the decision SHALL NOT be saved.
2. **AC-DECIDE-09** IF an employee without the manager or administrator role tries to approve their own record THEN
   the status SHALL NOT change AND the decision SHALL NOT be saved.
3. **AC-DECIDE-18** IF manager1@example.test is a responsible manager of their own main position AND tries to approve
   their own record with status "Awaiting decision" THEN the status SHALL NOT change AND manager1@example.test SHALL see
   "Your own record is decided by an administrator".
4. **AC-DECIDE-19** WHEN the administrator approves the record from AC-DECIDE-18 THEN the record SHALL have status
   "Approved".

### Requirement 4 — Employee changes to own records

**User Story:** As an employee, I want to correct, resubmit or cancel my records before they are decided, so that
managers only decide what I actually need.

#### Acceptance Criteria

1. **AC-DECIDE-06** WHEN a record has status "Returned for revision" AND the employee changes its hours from
   08:30–15:30 to 09:00–12:00 and resubmits it THEN the record SHALL have hours 09:00–12:00 and status "Awaiting
   decision" AND SHALL again be in the same manager's "Awaiting decision" list.
2. **AC-DECIDE-12** IF a record has status "Approved" AND the employee tries to change its hours THEN the change SHALL
   NOT be saved AND the employee SHALL see "This record is already approved and cannot be changed".
3. **AC-DECIDE-13** WHEN the employee cancels a record with status "Awaiting decision" THEN the record SHALL have status
   "Cancelled" AND SHALL disappear from the manager's "Awaiting decision" list.
4. **AC-DECIDE-16** WHEN a record 2026-11-02 08:30–15:30 has status "Awaiting decision" AND the employee changes its
   hours to 09:00–12:00 THEN the record SHALL have hours 09:00–12:00, SHALL keep status "Awaiting decision", AND the
   manager SHALL see 09:00–12:00 in the "Awaiting decision" list.
5. **AC-DECIDE-17** WHEN the employee cancels a record with status "Returned for revision" THEN the record SHALL have
   status "Cancelled" AND SHALL NOT be resubmittable.

## Business Rules from PM Assumptions

These are confirmed PM assumptions (`02-leave-decision.md`, "Assumptions"). They are not separate acceptance criteria
but every implementation and test must respect them.

- **BR-1 Statuses.** The five statuses "Awaiting decision", "Approved", "Rejected", "Returned for revision",
  "Cancelled" are final. IF a record is "Rejected" or "Cancelled" THEN the portal SHALL allow no further change or
  decision on it.
- **BR-2 Only "Awaiting decision" is decidable.** IF a record is not "Awaiting decision" THEN the portal SHALL refuse
  any decision on it and keep its status (AC-DECIDE-07 is the concurrent case of this rule).
- **BR-3 Own records.** A manager never decides their own records, even as the responsible manager of their own
  position; only an administrator decides them (AC-DECIDE-18, -19). A manager's own records therefore do not appear in
  their own "Awaiting decision" list.
- **BR-4 Positions without a manager.** IF a position has no responsible manager THEN its records SHALL be decidable
  by the administrator and SHALL appear in the administrator's "Awaiting decision" list.
- **BR-5 Employee changes.** An employee can change or cancel only their own record, and only while it is "Awaiting
  decision" or "Returned for revision"; resubmitting applies only to "Returned for revision". Changed hours must pass
  the same schedule and overlap rules as a new request (spec-1, AC-LEAVE-07..09, -13).
- **BR-6 Same managers.** A resubmitted or edited record goes to the same responsible managers of its position as the
  first time.
- **BR-7 Time.** All dates and times are Kyiv time (Europe/Kyiv); the decision time is stored and shown to the minute.
- **BR-8 Notifications.** No emails are sent; the employee learns of the decision from the status and comment in
  "My requests".

## Non-Functional Requirements

### Code Architecture and Modularity

- **Single Responsibility Principle:** decision rules, employee-change rules, read models and UI live in separate
  modules; each user action has exactly one server-side entry point.
- **Modular Design:** the decision entry point exposes an explicit extension point for checks made at approval time,
  so spec-3 can add the balance check without rewriting the decision rules.
- **Dependency Management:** this spec depends only on spec-1's data, helpers and screens; it does not change spec-1's
  rules for submitting requests.
- **Clear Interfaces:** every refusal is reported as a stable error code that maps to exactly one user-facing message.

### Performance

- The "Awaiting decision" and "History" lists and a record's history load in under 1 second for a school with up to
  500 employees and 20,000 records; lists are paginated server-side (page size 50).
- A decision is saved and both lists reflect it on the next page load (no manual refresh beyond navigation).

### Security (access control)

- Every rule of Requirement 3 and BR-3/BR-5 holds for direct API calls, not only in the UI (AC-DECIDE-08, -09, -18).
- A manager never sees records of positions they are not responsible for, even of the same employee (AC-DECIDE-01);
  the administrator sees all records.
- Only the record's employee can edit, resubmit or cancel it; managers and administrators cannot use these actions on
  other people's records in this slice (manager edits are WS11).
- Refusals reveal no data about records the caller cannot see.

### Reliability (concurrency)

- Decisions and employee changes on one record are serialised: the first completed action wins and the later one is
  refused with a message that names the current status (AC-DECIDE-07); there is never a lost update or a record with
  two decisions.
- Each action is atomic: the status change, its comment and its history line are saved together or not at all.

### Audit

- Every status change and every change of hours writes one history line with the date and time (to the minute), the
  person who acted, the action, the old and new status and any comment (AC-DECIDE-10, -15). History lines are never
  edited or deleted.
- The history is visible to the record's employee, every responsible manager of its position and the administrator.

### Usability

- Statuses and messages use the exact texts from the acceptance criteria and the glossary in `docs/product.md`.
- The "Awaiting decision" tab is the default view of the manager workspace; the record card shows employee, position,
  date, hours, leave type, the current status, the manager's comment and the history.
- Desktop-first; mobile layout is out of scope (`docs/product.md`).
