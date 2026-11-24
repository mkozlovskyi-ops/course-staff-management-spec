# Requirement: a manager submits a request on behalf of an employee and edits a record before deciding

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

An employee who falls ill in the morning often calls the manager instead of opening the portal, and the absence is then
recorded nowhere. A manager who sees a small mistake in a record (wrong date, wrong leave type) can today only return it
and wait. With this requirement the manager submits the request for the employee — their own positions' records are
approved at once — and corrects a record awaiting decision; the employee sees every change in the record's history.

### Users and roles

- Responsible manager (`manager`) — submits requests on behalf of employees of their positions; changes the date,
  hours, leave type or substitute of records of their positions with status "Awaiting decision".
- Administrator (`admin`) — does the same for any employee and any record.
- Employee (`employee`) — sees records submitted for them and every change in the record's history; cannot submit on
  behalf of anyone.
- A person who is not signed in sees nothing.

### Main scenario

1. The manager chooses an employee of their positions and fills in a request as the employee would: dates, positions,
   hours, leave type, substitute, comment.
2. The same checks as for the employee's own request apply: schedule, school calendar, overlaps, substitute.
3. Records of the positions this manager is responsible for are approved at once; records of other positions go to
   their managers with status "Awaiting decision". The history of each record shows who submitted it.
4. Before deciding, a manager may change the date, hours, leave type or substitute of a record of their positions; the
   history shows who changed what, from what, to what.
5. The employee sees the records and their history in "My requests".

### Acceptance criteria

- AC-BEHALF-01: Given manager1@example.test is the manager of teacher1@example.test's main position (schedule Mon–Fri 08:30–15:30). When manager1@example.test submits on behalf of teacher1@example.test "Sick leave" for the whole day Thursday 2026-12-03 for the main position. Then teacher1@example.test sees in "My requests" the record 2026-12-03 08:30–15:30 "Sick leave" with status "Approved", and its history has the line "Submitted by manager1@example.test on behalf of teacher1@example.test".
- AC-BEHALF-02: Given teacher1@example.test has the main position (manager manager1@example.test, Thu 08:30–15:30) and the additional position (manager manager2@example.test, Thu 15:45–17:00). When manager1@example.test submits on behalf of teacher1@example.test the whole day 2026-12-03 for both positions. Then the main-position record has status "Approved", and the additional-position record has status "Awaiting decision" and is in manager2@example.test's "Awaiting decision" list.
- AC-BEHALF-03: Given the employee and positions from AC-BEHALF-02. When the administrator submits on behalf of teacher1@example.test the whole day 2026-12-03 for both positions. Then both records have status "Approved".
- AC-BEHALF-04: Given the record from AC-BEHALF-01 and teacher1@example.test's "Sick leave" used 0.00 before it. When teacher1@example.test views their balances. Then "Sick leave" shows used 1.00, remaining 17.00.
- AC-BEHALF-05: Given teacher1@example.test's position has no Saturday in its schedule. When manager1@example.test tries to choose Saturday 2026-12-05 in a request on behalf of teacher1@example.test. Then the date is not accepted, and manager1@example.test sees the message "This employee has no working hours on this day".
- AC-BEHALF-06: Given teacher1@example.test has a record 2026-12-03 "Sick leave" 08:30–15:30 with status "Awaiting decision" of manager1@example.test's position. When manager1@example.test changes its date to Friday 2026-12-04. Then the record shows 2026-12-04 08:30–15:30 with status "Awaiting decision", and teacher1@example.test sees in its history the line "manager1@example.test changed the date: 2026-12-03 → 2026-12-04".
- AC-BEHALF-07: Given the record from AC-BEHALF-06 with type "Sick leave". When manager1@example.test changes its leave type to "Personal leave". Then the record shows "Personal leave", and its history has the line "manager1@example.test changed the leave type: Sick leave → Personal leave".
- AC-BEHALF-08: Given a record 2026-12-04 08:30–15:30 with status "Awaiting decision" and no substitute, and teacher2@example.test is on teacher1@example.test's substitute list. When manager1@example.test sets the substitute teacher2@example.test. Then the record shows the substitute teacher2@example.test, and its history has the line "manager1@example.test changed the substitute: No substitute → teacher2@example.test".
- AC-BEHALF-09: Given a record 2026-12-04 of a position with the schedule Fri 08:30–15:30 and status "Awaiting decision". When manager1@example.test changes its hours to 16:00–17:00. Then the change is not saved, and manager1@example.test sees the message "Hours must be within the schedule: 08:30–15:30".
- AC-BEHALF-10: Given a record of teacher1@example.test with status "Approved". When manager1@example.test tries to change its date. Then the change is not saved, and manager1@example.test sees the message "This record is already decided and cannot be changed".
- AC-BEHALF-11: Given teacher3@example.test has no positions managed by manager1@example.test. When manager1@example.test opens the request on behalf of teacher3@example.test by a direct link. Then manager1@example.test sees the message "You can submit requests only for employees of your positions", and no record of teacher3@example.test is created.
- AC-BEHALF-12: Given teacher1@example.test is signed in and has only the employee role. When teacher1@example.test opens the request on behalf of teacher2@example.test by a direct link. Then teacher1@example.test sees the message "Only a manager or an administrator can submit a request on behalf of an employee", and no record of teacher2@example.test is created.
- AC-BEHALF-13: Given a record of teacher1@example.test's main position with status "Awaiting decision", and manager2@example.test is not a manager of that position. When manager2@example.test tries to change its date directly, bypassing the portal's pages. Then the record keeps its date, hours, leave type and substitute, and its history has no new line.
- AC-BEHALF-14: Given nobody is signed in. When a visitor opens the request on behalf of an employee by a direct link. Then the visitor sees the sign-in page, and no names or records are shown.
- AC-BEHALF-15: Given manager3@example.test is a manager but no position is assigned to them. When manager3@example.test opens the request on behalf of an employee. Then manager3@example.test sees the message "You have no employees to submit requests for".

### Out of scope

- A manager submitting on behalf of themselves: that is their own request (`01-leave-request.md`), decided by an
  administrator (AC-DECIDE-18).
- Changing a record that is already decided: cancellation is in `09-cancel-approved.md`.
- The employee agreeing to the manager's change: the change applies at once and is shown in the history.
- Changing the position of a record: cancel the record and submit a new one.
- Notifying the employee by email: only the history and "My requests".

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv).
- Employees, positions, managers and substitute lists for testing are created as test data or by the administrator
  (`07-staff-management.md`); `manager3@example.test` has the manager role and no positions.
- A request on behalf follows every rule of `01-leave-request.md`, `03-leave-balance.md`, `04-school-calendar.md` and
  `06-substitutes.md`; messages that say "you" / "your" for the employee say "this employee" for the manager
  (AC-BEHALF-05). The over-balance warning is shown to the manager; a record over the balance is approved with the
  suggested split, as in AC-BALANCE-10.
- "Approved at once" applies only to records of positions the submitting manager is responsible for (or is a deputy for
  on that day); the history of such a record shows both "Submitted by … on behalf of …" and "Approved by …" with the
  same time.
- Changing the date or hours runs the same checks as a new record: schedule, school calendar, overlap with the
  employee's other records (the record itself is not checked against itself), the substitute's availability.
- Every change adds one history line per changed field: who, when (to the minute), field, old value → new value.
- No amounts of money anywhere in this requirement.

### Open questions

None.

### Change log

- 2026-11-24 — created; part of roadmap row WS11 — PM's own decisions: on-behalf records of the manager's own positions are approved at once, other positions go to their managers; changes before deciding apply at once and are shown in the history
- 2026-11-24 — Checked against the checklist: items 1–16 yes. Refusals of two kinds: forbidden role (AC-BEHALF-11 other employees, AC-BEHALF-12 employee, AC-BEHALF-13 bypassing the pages) and not signed in (AC-BEHALF-14); interface state: no employees to submit for (AC-BEHALF-15). Kept on purpose: 15 ACs. AI chat: no.
