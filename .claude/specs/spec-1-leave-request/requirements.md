Spec 1 of 3 — leave request. Depends on: none. Used by: spec-2-leave-decision, spec-3-leave-balance.

# Requirements Document

> Source: `docs/requirements/01-leave-request.md` and `docs/product.md` (PM repo, 2026-10-19), split out of the
> combined spec `.claude/specs/leave-management-ws6/`. Every acceptance criterion keeps the PM's AC ID so tests and
> conformance tables can be named after it. If this document and the PM text disagree, the PM text wins.

## Introduction

Spec 1 of slice WS6 of the school staff leave portal. An employee requests leave (whole day or specific hours) for one
or more dates and one or more of their positions in a single request. The portal checks the hours against each
position's schedule, shows a summary, and on confirmation creates one leave record per included position per date, each
with status "Awaiting decision". The employee then sees their own records and statuses in "My requests".

Today staff ask for leave verbally or in spreadsheets, and an employee with two positions has to arrange it with two
managers separately. This feature replaces that with one request whose records go to each position's own manager.

This spec also lays the shared foundation used by the other two specs of WS6: portal roles, divisions, positions,
responsible managers, schedules, leave types, requests and records.

Not in this spec:

- Managers' decisions, the manager workspace, and the employee editing, resubmitting or cancelling a record — see
  spec-2-leave-decision.
- Balances, day shares and the over-balance warning and split — see spec-3-leave-balance.

## Alignment with Product Vision

- Success indicator "An employee requests leave for several dates and both of their positions in one go, without
  asking a manager verbally" → Requirements 1–3.
- "Each manager sees only the records of their own positions" → this spec creates one record per position so that
  spec-2 can route each record to that position's responsible manager.
- "Balances … match the approved records" → this spec stores each record's exact minutes and the employee's scheduled
  minutes for that date so that spec-3 can count day shares.
- Permission matrix (`docs/product.md`): every role (employee, manager, admin) may submit their own request and see their
  own requests; submitting on behalf of another employee is slice WS11 and out of scope.

## Requirements

### Requirement 1 — Submit a leave request

**User Story:** As an employee, I want to request leave for one or more dates and choose, per position, a whole day or
specific hours within my schedule, so that each of my managers knows exactly when I am absent from their position.

#### Acceptance Criteria

1. **AC-LEAVE-01** WHEN an employee with main position "Elementary school / Teacher" (schedule Mon–Fri 08:30–15:30)
   requests "Sick leave" for the whole day on Monday 2026-11-02 and confirms the summary THEN the portal SHALL create 1
   leave record: position "Elementary school / Teacher", 2026-11-02, 08:30–15:30, type "Sick leave", status "Awaiting
   decision".
2. **AC-LEAVE-02** WHEN an employee with main position "Elementary school / Teacher" (Mon 08:30–15:30) and additional
   position "High school / Math teacher" (Mon 15:45–17:00) requests the whole day 2026-11-02 with both positions
   included THEN the portal SHALL create 2 records in one request — 08:30–15:30 for the main position and 15:45–17:00
   for the additional position — both with status "Awaiting decision".
3. **AC-LEAVE-03** WHEN the employee with two positions from AC-LEAVE-02 requests 2026-11-02 and excludes the
   additional position THEN the portal SHALL create only 1 record — for the main position — AND SHALL create no record
   for the additional position.
4. **AC-LEAVE-04** WHEN an employee with a main position (schedule Mon–Fri 08:30–15:30) requests "Personal leave" on
   2026-11-02 from 09:00 to 11:30 THEN the portal SHALL create 1 record 2026-11-02 09:00–11:30 with a duration of
   2 h 30 min.
5. **AC-LEAVE-05** WHEN an employee with a main position (schedule Mon–Fri 08:30–15:30) requests the dates 2026-11-02
   and 2026-11-03, whole day THEN the portal SHALL create 2 records (one per date), both belonging to the same request.
6. **AC-LEAVE-15** WHEN an employee whose position has the schedule blocks Tue 08:30–12:00 and 12:30–15:30 requests
   "Personal leave" on 2026-11-03 from 11:00 to 13:00 THEN the portal SHALL create 1 record 2026-11-03 11:00–13:00 with
   a duration of 1 h 30 min (the gap 12:00–12:30 is not counted).
7. **AC-LEAVE-16** IF the employee already has a main-position record 2026-11-03 09:00–11:00 with status "Awaiting
   decision" AND confirms a whole-day request for 2026-11-02 and 2026-11-03 THEN the portal SHALL create no record for
   either date AND SHALL show "You already have a request for this time: 2026-11-03 09:00–11:00".

### Requirement 2 — Validate request input

**User Story:** As an employee, I want clear messages when my request is not possible, so that I can correct it
before submitting.

#### Acceptance Criteria

1. **AC-LEAVE-06** IF an employee whose position has no Saturday in its schedule tries to choose Saturday 2026-11-07
   THEN the portal SHALL not accept the date AND SHALL show "You have no working hours on this day".
2. **AC-LEAVE-07** IF a position has the schedule block 08:30–13:00 AND the employee enters the hours 08:00–10:00 THEN
   the portal SHALL create no request AND SHALL show "Hours must be within your schedule: 08:30–13:00".
3. **AC-LEAVE-08** IF a position has the schedule block 08:30–13:00 AND the employee enters 13:00 as the start THEN the
   portal SHALL create no request AND SHALL show "13:00 is the end of working time and can only be chosen as the end of
   leave".
4. **AC-LEAVE-09** IF a position has the schedule 08:30–15:30 AND the employee enters the hours from 11:00 to 10:00
   THEN the portal SHALL create no request AND SHALL show "The end must be later than the start".
5. **AC-LEAVE-10** IF the employee chose the date 2026-11-02 and excluded all positions AND tries to submit the request
   THEN the portal SHALL create no request AND SHALL show "Choose at least one position".
6. **AC-LEAVE-12** IF the employee chose the type "Family celebration" AND submits the request without stating the
   relationship THEN the portal SHALL create no request AND SHALL show "State your relationship to this person:
   sibling, child, parent, grandchild or grandparent. Not applicable to cousins, friends, neighbours or
   siblings-in-law".
7. **AC-LEAVE-13** IF the employee already has a main-position record 2026-11-02 09:00–11:00 with status "Awaiting
   decision" AND requests the same position on 2026-11-02 10:00–12:00 THEN the portal SHALL create no request AND SHALL
   show "You already have a request for this time: 2026-11-02 09:00–11:00".

### Requirement 3 — Confirm before saving

**User Story:** As an employee, I want to review a summary of dates, hours, positions and leave type before anything is
saved, so that I do not submit leave by mistake.

#### Acceptance Criteria

1. **AC-LEAVE-11** WHEN the employee has filled in a request for 2 dates, sees the summary, does not confirm it and goes
   back to editing THEN the portal SHALL create no record AND SHALL keep the entered dates and hours for editing.

### Requirement 4 — My requests

**User Story:** As an employee, I want to see all my own leave records and the status of each, so that I know what I
have requested and what has been decided.

#### Acceptance Criteria

1. **AC-LEAVE-14** WHEN teacher1@example.test, who has 2 records, opens "My requests" while teacher2@example.test has 3
   records THEN the list SHALL contain exactly 2 records — only their own — each with date, hours, position, leave type
   and status.

In this spec "My requests" is read-only. Showing the manager's comment and the edit, resubmit and cancel actions are
added by spec-2-leave-decision.

## Business Rules and Edge Cases

These rules come from the PM's "Assumptions" section. They carry no AC ID of their own; each is exercised by the ACs
listed next to it and by the tests in `tasks.md`.

1. **BR-1 Time zone.** All dates and times SHALL be entered, stored and shown in Kyiv time (Europe/Kyiv), the same for
   all users, regardless of the browser's time zone.
2. **BR-2 Time precision.** Times SHALL be chosen with 5-minute precision, and duration SHALL be counted in minutes
   (AC-LEAVE-04, -15).
3. **BR-3 Block edges.** A schedule block SHALL include its start time and exclude its end time: in the block
   08:30–13:00 the latest possible leave start is 12:55; the end may equal a block's end (AC-LEAVE-07, -08).
4. **BR-4 Whole day.** "Whole day" for a position SHALL mean all blocks of its schedule on that date (AC-LEAVE-01,
   -02, -05).
5. **BR-5 Gaps.** A request MAY span a gap between schedule blocks; only the minutes inside blocks SHALL count. The
   start and the end SHALL each fall inside a block (the end may equal a block's end) (AC-LEAVE-15).
6. **BR-6 Working day.** In this slice a working day for a position SHALL be defined only by that position's schedule;
   holidays, school breaks and early dismissals are slice WS7 (AC-LEAVE-06).
7. **BR-7 All or nothing.** A request SHALL be saved as a whole: either all of its records are created or none
   (AC-LEAVE-16).
8. **BR-8 One type per request.** A request SHALL have exactly one leave type; different types mean different requests.
9. **BR-9 Overlaps.** Records of the same position SHALL NOT overlap in time while their status is "Awaiting decision",
   "Returned for revision" or "Approved"; records with status "Rejected" or "Cancelled" SHALL NOT block a new request
   (AC-LEAVE-13, -16).
10. **BR-10 Past and future dates.** An employee MAY request a date in the past (for example, sick leave after the
    fact); there SHALL be no limit on how many days ahead or back a request can be made.
11. **BR-11 Own requests only.** Every user (employee, manager, admin) SHALL submit requests only on their own behalf
    and only for their own positions; submitting on behalf of another employee is slice WS11.
12. **BR-12 Leave types.** The leave types SHALL be "Sick leave", "Personal leave", "Unpaid leave" and "Family
    celebration"; the relationship for "Family celebration" SHALL be one of sibling, child, parent, grandchild or
    grandparent (AC-LEAVE-12).

## Out of Scope

- Holidays, school breaks and early dismissals in the school calendar — slice WS7.
- Choosing a substitute and the assistant upgrade — slice WS8.
- A manager or administrator submitting a request on behalf of an employee — slice WS11.
- The reminder to talk to the administrator about Personal leave of 2 or more days — slice WS11.
- An employee cancelling an approved record — slice WS11.
- Decisions, the manager workspace, and editing, resubmitting or cancelling own records — see spec-2-leave-decision.
- Balances, day shares and the over-balance warning — see spec-3-leave-balance.
- Managing employees, positions, schedules and managers — slice WS10; test data is created by the developer.

## Non-Functional Requirements

### Code Architecture and Modularity

- **Single Responsibility Principle:** working-hour rules (schedule blocks, interval validation, minutes inside blocks)
  live in pure functions with no I/O, separate from data access and UI.
- **Modular Design:** schedule resolution, error messages, the submit write path and the "My requests" read path are
  separate modules; specs 2 and 3 extend them without rewriting them.
- **Dependency Management:** UI depends on server actions; server actions depend on domain functions and the database
  client; domain functions depend on nothing.
- **Clear Interfaces:** creating a request goes through a single server-side write path, so the rules hold for direct
  API calls too; every user-facing error is identified by a stable code mapped to one exact message.

### Performance

- Submitting a request of up to 10 dates × 3 positions SHALL complete in under 1 second on the local stack.
- "My requests" SHALL be paginated server-side with a page size of 50 and render the first page in under 1 second for
  an employee with 500 records.

### Security

- Access rules are enforced by the data layer for every new kind of data, not only in the UI; in "My requests" a user
  SHALL see only their own records (AC-LEAVE-14), including when they also hold the manager or admin role; an employee
  SHALL see only their own positions and schedules.
- The employee of a new request SHALL always be the signed-in user; a request for someone else's position SHALL be
  refused, also for direct API calls (BR-11).
- Nobody can create, change or delete requests and records except through the submit path.
- No salary or pay-rate data is stored.
- Every created record gets an event entry saying who created it and when.

### Reliability

- A request is created in one transaction: all records or none (AC-LEAVE-16, BR-7).
- Overlaps are prevented by the database itself, so two simultaneous submissions cannot both create overlapping
  records (AC-LEAVE-13, BR-9).
- The same validation rules are applied in the form (for instant feedback) and on the server (for enforcement); the
  server decision is final.

### Usability

- Every validation error uses the exact message text from the AC, shown next to the date or position row it concerns.
- Times are chosen in 5-minute steps and shown in Europe/Kyiv (BR-1, BR-2); durations are shown as "X h Y min".
- The date picker offers only dates on which at least one of the employee's positions has working hours (AC-LEAVE-06).
- Going back from the summary keeps everything entered (AC-LEAVE-11).
- Desktop-first; mobile layout is out of scope (`docs/product.md`).
