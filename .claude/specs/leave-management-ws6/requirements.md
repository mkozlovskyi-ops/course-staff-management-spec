# Requirements Document

> Source: `docs/requirements/01-leave-request.md`, `02-leave-decision.md`, `03-leave-balance.md` and `docs/product.md`
> (PM repo, 2026-10-19). Every acceptance criterion keeps the PM's AC ID so tests and conformance tables can be named
> after it. If this document and the PM text disagree, the PM text wins.

## Introduction

Slice WS6 of the school staff leave portal. An employee requests leave (whole day or specific hours) for one or more
dates and one or more of their positions; the portal creates one leave record per position per date. The responsible
manager of each position decides on each record separately (approve, reject, return for revision). The portal counts
Sick leave and Personal leave balances in days from approved records, and treats any excess as Unpaid leave after the
manager approves a suggested split.

## Alignment with Product Vision

- "An employee requests leave for several dates and both positions in one go" → Requirements 1–4.
- "Each manager sees only the records of their own positions and decides on each separately" → Requirements 5–8.
- "Balances are calculated automatically to 0.01 day and match approved records" → Requirements 9–11.
- The register, school calendar, substitutes and admin management are later slices (WS7–WS11) and are out of scope.

## Requirements

### Requirement 1 — Submit a leave request

**User Story:** As an employee, I want to request leave for one or more dates and choose, per position, a whole day or
specific hours, so that each of my managers knows exactly when I am absent from their position.

#### Acceptance Criteria

1. **AC-LEAVE-01** WHEN an employee with main position "Elementary school / Teacher" (Mon–Fri 08:30–15:30) requests
   "Sick leave" for the whole day 2026-11-02 and confirms THEN the portal SHALL create 1 record: that position,
   2026-11-02, 08:30–15:30, "Sick leave", status "Awaiting decision".
2. **AC-LEAVE-02** WHEN an employee with a main position (Mon 08:30–15:30) and an additional position
   (Mon 15:45–17:00) requests the whole day 2026-11-02 with both positions included THEN the portal SHALL create 2
   records in one request (08:30–15:30 and 15:45–17:00), both "Awaiting decision".
3. **AC-LEAVE-03** WHEN the employee from AC-LEAVE-02 requests 2026-11-02 with the additional position excluded THEN
   the portal SHALL create only the main-position record.
4. **AC-LEAVE-04** WHEN an employee requests "Personal leave" on 2026-11-02 from 09:00 to 11:30 on a position
   scheduled 08:30–15:30 THEN the portal SHALL create 1 record 09:00–11:30 with a duration of 150 minutes.
5. **AC-LEAVE-05** WHEN an employee requests 2026-11-02 and 2026-11-03, whole day THEN the portal SHALL create 2
   records, one per date, belonging to the same request.
6. **AC-LEAVE-15** WHEN an employee requests 11:00–13:00 on a date whose schedule blocks are 08:30–12:00 and
   12:30–15:30 THEN the portal SHALL create 1 record 11:00–13:00 with a duration of 90 minutes (the gap is not counted).
7. **AC-LEAVE-16** IF any record of a request is invalid (for example, it overlaps an existing record on 2026-11-03)
   THEN the portal SHALL create no record for any date of that request AND SHALL show "You already have a request
   for this time: 2026-11-03 09:00–11:00".

### Requirement 2 — Validate request input

**User Story:** As an employee, I want clear messages when my request is not possible, so that I can correct it
before submitting.

#### Acceptance Criteria

1. **AC-LEAVE-06** IF the employee chooses a date on which none of their positions is scheduled (Saturday 2026-11-07)
   THEN the portal SHALL not accept the date AND SHALL show "You have no working hours on this day".
2. **AC-LEAVE-07** IF the chosen hours fall outside the schedule block 08:30–13:00 (for example 08:00–10:00) THEN the
   portal SHALL create no request AND SHALL show "Hours must be within your schedule: 08:30–13:00".
3. **AC-LEAVE-08** IF the chosen start equals the end of a block (13:00 in 08:30–13:00) THEN the portal SHALL create no
   request AND SHALL show "13:00 is the end of working time and can only be chosen as the end of leave".
4. **AC-LEAVE-09** IF the end is not later than the start (11:00 → 10:00) THEN the portal SHALL create no request AND
   SHALL show "The end must be later than the start".
5. **AC-LEAVE-10** IF all positions of the chosen date are excluded THEN the portal SHALL create no request AND SHALL
   show "Choose at least one position".
6. **AC-LEAVE-12** IF the leave type is "Family celebration" and no relationship is given THEN the portal SHALL
   create no request AND SHALL show the relationship message listing sibling, child, parent, grandchild, grandparent
   and the exclusions (cousins, friends, neighbours, siblings-in-law).
7. **AC-LEAVE-13** IF the requested interval overlaps a record of the same position with status "Awaiting decision",
   "Returned for revision" or "Approved" THEN the portal SHALL create no request AND SHALL show "You already have a
   request for this time: <date> <start>–<end>".

### Requirement 3 — Confirm before saving

**User Story:** As an employee, I want to review a summary before anything is saved, so that I do not submit leave by
mistake.

#### Acceptance Criteria

1. **AC-LEAVE-11** WHEN the employee sees the summary of a 2-date request and goes back to editing instead of
   confirming THEN the portal SHALL create no record AND SHALL keep the entered dates and hours for editing.

### Requirement 4 — My requests

**User Story:** As an employee, I want to see all my records and their statuses, so that I know which leave is
approved.

#### Acceptance Criteria

1. **AC-LEAVE-14** WHEN teacher1@example.test (2 records) opens "My requests" while teacher2@example.test has 3 records
   THEN the portal SHALL list exactly the 2 own records, each with date, hours, position, leave type and status.

### Requirement 5 — Manager queue

**User Story:** As a responsible manager, I want a list of records awaiting my decision for my positions only, so that
I can process them without seeing other divisions' data.

#### Acceptance Criteria

1. **AC-DECIDE-01** WHEN manager1@example.test (main position) opens "Awaiting decision" for an employee whose
   additional position belongs to manager2@example.test THEN the list SHALL contain the main-position record AND
   SHALL NOT contain the additional-position record.
2. **AC-DECIDE-11** WHEN the manager approves 1 of 3 records in "Awaiting decision" THEN 2 records SHALL remain there
   AND the approved one SHALL appear in the manager's "History".
3. **AC-DECIDE-14** WHEN a manager with 0 records awaiting decision opens the list THEN the portal SHALL show
   "No records awaiting decision".

### Requirement 6 — Decide on a record

**User Story:** As a responsible manager, I want to approve, reject or return each record separately, so that one
decision never changes the employee's other records.

#### Acceptance Criteria

1. **AC-DECIDE-02** WHEN the manager approves one of 2 records of a request THEN that record SHALL be "Approved" AND
   the other SHALL stay "Awaiting decision".
2. **AC-DECIDE-03** WHEN the manager rejects a record THEN its status SHALL be "Rejected" AND the employee SHALL see it
   in "My requests".
3. **AC-DECIDE-04** IF the manager returns a record without a comment THEN the status SHALL NOT change AND the portal
   SHALL show "Tell the employee what to fix".
4. **AC-DECIDE-05** WHEN the manager returns a record with the comment "Enter hours instead of a whole day" THEN the
   status SHALL be "Returned for revision" AND the employee SHALL see that comment on the record.
5. **AC-DECIDE-07** WHEN a position has 2 responsible managers AND the first approves AND the second then tries to
   reject THEN the record SHALL stay "Approved" AND the second manager SHALL see "Another manager has already
   decided: Approved".
6. **AC-DECIDE-10** WHEN the administrator approves a record of a position they do not manage THEN the record SHALL be
   "Approved" AND its history SHALL show the administrator as the decider.
7. **AC-DECIDE-15** WHEN the employee or any responsible manager of the position views an approved record's history
   THEN it SHALL show a line with the decision date and time, who decided, and "Approved".

### Requirement 7 — Decision access control

**User Story:** As the school, I want only the right people to decide records, so that nobody approves their own or
someone else's leave.

#### Acceptance Criteria

1. **AC-DECIDE-08** IF manager2@example.test, who is not responsible for the position, tries to approve its record THEN
   the status SHALL NOT change AND the decision SHALL NOT be saved.
2. **AC-DECIDE-09** IF an employee without the manager or admin role tries to approve their own record THEN the status
   SHALL NOT change AND the decision SHALL NOT be saved.
3. **AC-DECIDE-18** IF a manager tries to approve their own record of a position they manage THEN the status SHALL NOT
   change AND the portal SHALL show "Your own record is decided by an administrator".
4. **AC-DECIDE-19** WHEN the administrator approves the record from AC-DECIDE-18 THEN the record SHALL be "Approved".

### Requirement 8 — Employee changes to own records

**User Story:** As an employee, I want to correct or withdraw my records before they are decided, so that managers
only decide what I actually need.

#### Acceptance Criteria

1. **AC-DECIDE-06** WHEN the employee changes a returned record from 08:30–15:30 to 09:00–12:00 and resubmits THEN the
   record SHALL have 09:00–12:00, status "Awaiting decision", AND SHALL be in the same manager's list again.
2. **AC-DECIDE-12** IF the employee tries to change the hours of an "Approved" record THEN the change SHALL NOT be saved
   AND the portal SHALL show "This record is already approved and cannot be changed".
3. **AC-DECIDE-13** WHEN the employee cancels a record "Awaiting decision" THEN it SHALL be "Cancelled" AND SHALL leave
   the manager's "Awaiting decision" list.
4. **AC-DECIDE-16** WHEN the employee changes a record "Awaiting decision" from 08:30–15:30 to 09:00–12:00 THEN it SHALL
   keep status "Awaiting decision" AND the manager SHALL see 09:00–12:00.
5. **AC-DECIDE-17** WHEN the employee cancels a record "Returned for revision" THEN it SHALL be "Cancelled" AND SHALL NOT
   be resubmittable.

### Requirement 9 — Balance calculation and view

**User Story:** As an employee, I want to see granted, carried-over, used and remaining days per leave type, so that I
know how much leave I have left.

#### Acceptance Criteria

1. **AC-BALANCE-01** WHEN an employee with no approved records and no carry-over in 2026–2027 views balances THEN the
   portal SHALL show Sick leave and Personal leave as granted 18.00 / used 0.00 / remaining 18.00, and Unpaid leave and
   Family celebration as used 0.00 only.
2. **AC-BALANCE-02** WHEN a whole-day "Sick leave" record (Tue 08:30–15:30) is approved THEN used SHALL be 1.00 and
   remaining 17.00.
3. **AC-BALANCE-03** WHEN a "Sick leave" record 09:00–11:30 on a 420-minute day is approved THEN used SHALL be 0.36 and
   remaining 17.64.
4. **AC-BALANCE-04** WHILE a whole-day "Personal leave" record is "Awaiting decision" the balance SHALL show used 0.00,
   remaining 18.00.
5. **AC-BALANCE-05** WHEN both whole-day "Personal leave" records of a date with two positions are approved THEN used
   SHALL be 1.00, not 2.00.
6. **AC-BALANCE-06** WHEN a whole-day record for only the additional position (75 of 495 minutes that date) is approved
   THEN used SHALL be 0.15.
7. **AC-BALANCE-07** WHEN a whole-day "Sick leave" record "Awaiting decision" is rejected THEN Sick leave SHALL show used
   0.00, remaining 18.00.
8. **AC-BALANCE-11** WHEN an employee with 5.00 Sick leave days left on 2027-08-31 views balances on 2027-09-01 THEN
   Sick leave 2027–2028 SHALL show granted 18.00, carried over 5.00, remaining 23.00.

### Requirement 10 — Over-balance warning and split

**User Story:** As an employee and a manager, I want to be warned when a request exceeds the balance and to have the
excess counted as Unpaid leave, so that paid leave is never overspent.

#### Acceptance Criteria

1. **AC-BALANCE-08** WHEN an employee with Personal leave remaining 0.50 requests a whole day of Personal leave THEN the
   portal SHALL create the request AND SHALL show "Your Personal leave balance is 0.50 day. Another 0.50 day will be
   counted as Unpaid leave".
2. **AC-BALANCE-15** WHEN an employee with Personal leave remaining 0.20 requests 09:00–11:30 (0.36 day) THEN the portal
   SHALL create the request AND SHALL show "Your Personal leave balance is 0.20 day. Another 0.16 day will be counted as
   Unpaid leave".
3. **AC-BALANCE-09** WHEN the manager opens a whole-day Personal leave record of an employee with used 17.50 /
   remaining 0.50 THEN the portal SHALL show "This request exceeds the Personal leave balance (0.50 day). Suggested:
   0.50 day Personal leave, 0.50 day Unpaid leave".
4. **AC-BALANCE-14** WHEN, with remaining 1.50 and two whole-day Personal records awaiting decision, the manager approves
   the first and then opens the second THEN the portal SHALL show the warning computed from remaining 0.50 (0.50
   Personal + 0.50 Unpaid).
5. **AC-BALANCE-10** WHEN the manager approves the AC-BALANCE-09 record with the split (Unpaid used 0.00 before) THEN
   Personal leave SHALL show used 18.00 / remaining 0.00 AND Unpaid leave used 0.50.
6. **AC-BALANCE-16** IF the manager tries to approve the AC-BALANCE-09 record entirely as Personal leave THEN the record
   SHALL NOT be approved AND the portal SHALL show "The Personal leave balance is 0.50 day: approve with the suggested
   split or reject".

### Requirement 11 — Balance visibility

**User Story:** As an employee, I want my balances to be private, so that only my managers and the administrator see
them.

#### Acceptance Criteria

1. **AC-BALANCE-12** IF manager1@example.test manages none of teacher1@example.test's positions and tries to view their
   balances THEN the portal SHALL NOT show them.
2. **AC-BALANCE-13** IF teacher1@example.test tries to view another employee's balances THEN the portal SHALL NOT show
   them.

## Non-Functional Requirements

### Code Architecture and Modularity

- **Single Responsibility:** leave-duration and balance arithmetic live in pure functions with no I/O, separate from data
  access and UI.
- **Modular Design:** schedule resolution, request validation, decisions and balances are separate modules.
- **Dependency Management:** UI depends on server actions; server actions depend on domain functions and the database
  client; domain functions depend on nothing.
- **Clear Interfaces:** every state change goes through one database function per action, so rules hold for direct API
  calls too.

### Performance

- "Awaiting decision", "History" and "My requests" lists are paginated server-side (page size 50).
- Balance for one employee and one school year is computed in one query.

### Security

- Row Level Security on every table; every rule from the permission matrix in `docs/product.md` holds for direct API
  calls, not only in the UI (AC-DECIDE-08, -09, -18, AC-BALANCE-12, -13).
- No salary or pay-rate data is stored.
- Every decision is recorded with who, what and when (AC-DECIDE-15).

### Reliability

- A request is created in one transaction: all records or none (AC-LEAVE-16).
- Concurrent decisions on one record: the first wins, the second gets "Another manager has already decided"
  (AC-DECIDE-07).
- The balance check at approval time uses the balance at that moment (AC-BALANCE-14).

### Usability

- All dates and times are shown in Europe/Kyiv regardless of the browser's time zone; time is chosen in 5-minute steps.
- Day values are shown with 2 decimals; every validation error uses the exact message text from the AC.
- Desktop-first; mobile layout is out of scope.
