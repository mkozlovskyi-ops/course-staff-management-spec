# Requirement: a manager decides on leave records of their positions

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Today a manager learns about an employee's absence from a conversation or a spreadsheet and does not always know
whether it concerns their division. When an employee has two positions with different managers, it is unclear who
approves. With this requirement each manager sees only the records of their own positions, decides on each record
separately, and the employee sees the result and the comment.

### Users and roles

- Responsible manager (`manager`) — sees and decides only records of the positions they are assigned to. Does not see
  records of other positions, even of the same employee.
- Administrator (`admin`) — sees and can decide any record.
- Employee (`employee`) — sees the decision and comment on their own records; corrects a returned record and resubmits
  it. Makes no decisions.

### Main scenario

1. The manager opens the "Awaiting decision" list — records of the positions they are responsible for.
2. The manager reviews a record: employee, position, date, hours, leave type, employee's comment.
3. The manager makes one of the decisions: approve, reject, or return for revision with a comment.
4. The record leaves the "Awaiting decision" list and appears in the manager's "History".
5. The employee sees the new record status and the manager's comment in "My requests".
6. If the record was returned, the employee corrects it and resubmits — the record is again in the "Awaiting
   decision" list of the same managers.

### Acceptance criteria

- AC-DECIDE-01: Given an employee with a main position (manager manager1@example.test) and an additional position (manager manager2@example.test) requested a whole day with both positions. When manager1@example.test opens the "Awaiting decision" list. Then it contains this employee's main-position record and does not contain the additional-position record.
- AC-DECIDE-02: Given a request with 2 records with status "Awaiting decision". When the manager approves one of them. Then that record has status "Approved", and the other stays "Awaiting decision".
- AC-DECIDE-03: Given a record with status "Awaiting decision". When the manager rejects it. Then the record has status "Rejected", and the employee sees this status in "My requests".
- AC-DECIDE-04: Given a record with status "Awaiting decision". When the manager returns it for revision without a comment. Then the status does not change, and the manager sees the message "Tell the employee what to fix".
- AC-DECIDE-05: Given a record with status "Awaiting decision". When the manager returns it for revision with the comment "Enter hours instead of a whole day". Then the record has status "Returned for revision", and the employee sees the comment "Enter hours instead of a whole day" on it.
- AC-DECIDE-06: Given a record with status "Returned for revision". When the employee changes the hours from 08:30–15:30 to 09:00–12:00 and resubmits the record. Then the record has hours 09:00–12:00, status "Awaiting decision", and is again in the same manager's "Awaiting decision" list.
- AC-DECIDE-07: Given a position has 2 responsible managers, and both see its record with status "Awaiting decision". When the first manager approves the record and the second then tries to reject it. Then the record stays "Approved", and the second manager sees the message "Another manager has already decided: Approved".
- AC-DECIDE-08: Given manager2@example.test is not responsible for the employee's main position. When manager2@example.test tries to approve a record of that position. Then the status does not change, and the decision is not saved.
- AC-DECIDE-09: Given an employee without the manager or administrator role. When this employee tries to approve their own record. Then the status does not change, and the decision is not saved.
- AC-DECIDE-10: Given a record with status "Awaiting decision" of a position for which the administrator is not a responsible manager. When the administrator approves this record. Then the record has status "Approved", and the record's history shows that the administrator made the decision.
- AC-DECIDE-11: Given the manager's "Awaiting decision" list has 3 records. When the manager approves 1 of them. Then 2 records remain in "Awaiting decision", and the approved record is in the manager's "History".
- AC-DECIDE-12: Given a record with status "Approved". When the employee tries to change its hours. Then the change is not saved, and the employee sees the message "This record is already approved and cannot be changed".
- AC-DECIDE-13: Given a record with status "Awaiting decision". When the employee cancels it. Then the record has status "Cancelled" and disappears from the manager's "Awaiting decision" list.
- AC-DECIDE-14: Given the manager has 0 records with status "Awaiting decision". When the manager opens the "Awaiting decision" list. Then the manager sees the message "No records awaiting decision".
- AC-DECIDE-15: Given a manager approved a record. When the employee or any responsible manager of the position views the record's history. Then the history has a line: date and time of the decision, who decided, decision "Approved".
- AC-DECIDE-16: Given a record 2026-11-02 08:30–15:30 with status "Awaiting decision". When the employee changes its hours to 09:00–12:00. Then the record has hours 09:00–12:00, keeps status "Awaiting decision", and the manager sees 09:00–12:00 in the "Awaiting decision" list.
- AC-DECIDE-17: Given a record with status "Returned for revision". When the employee cancels it. Then the record has status "Cancelled" and cannot be resubmitted.
- AC-DECIDE-18: Given manager1@example.test is a responsible manager of their own main position and has an own record with status "Awaiting decision". When manager1@example.test tries to approve that record. Then the status does not change, and manager1@example.test sees the message "Your own record is decided by an administrator".
- AC-DECIDE-19: Given the record from AC-DECIDE-18. When the administrator approves it. Then the record has status "Approved".
- AC-DECIDE-20: Given a record of a request with the comment "Doctor's appointment" and status "Awaiting decision". When the manager opens the record. Then the manager sees the comment "Doctor's appointment".
- AC-DECIDE-21: Given a record 2026-11-02 "Sick leave" with status "Awaiting decision". When the employee tries to change its leave type to "Personal leave". Then the change is not saved, and the employee sees the message "Only the hours can be changed. To change the date, position or leave type, cancel this record and submit a new request".
- AC-DECIDE-22: Given two records of the same position on 2026-11-02, 08:30–10:00 and 13:00–15:30, both with status "Awaiting decision". When the employee changes the second record to 09:00–15:30. Then the change is not saved, and the employee sees the message "You already have a request for this time: 2026-11-02 08:30–10:00".

### Out of scope

- A manager changing the date, hours, leave type or substitute before deciding — a later slice (WS11).
- Cancelling an approved record (by the employee through the manager, or by the manager with a note) — slice WS11.
- A deputy manager while the manager is absent — slice WS10.
- The warning to the manager about exceeding the balance — requirement `03-leave-balance.md`.
- The warning about a conflict with the school calendar — slice WS7.
- Emails to the manager or the employee: only the status and an in-app marker.
- Approving all records of one request in a single action.

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv); the decision time is stored to the minute.
- Positions and responsible managers for testing are created by the developer as test data; managing them is slice
  WS10.
- The list of 5 statuses ("Awaiting decision", "Approved", "Rejected", "Returned for revision", "Cancelled") is
  final (confirmed by PM).
- Statuses "Rejected" and "Cancelled" are final: a record does not move anywhere from them.
- A manager never decides their own records, even as the responsible manager of their own position: only an
  administrator decides them (confirmed by PM).
- Records of a position that has no responsible manager left are handled by the administrator (confirmed by PM;
  flagging such positions is slice WS10).
- An employee can cancel only their own record with status "Awaiting decision" or "Returned for revision".
- A resubmitted record goes to the same responsible managers of the position as the first time.
- Before a decision the employee can change only the hours of a record, within the schedule of the same position on
  the same date; changing the date, position or leave type means cancelling the record and submitting a new request.
- When the employee changes hours, the record is not checked for overlap against itself, only against the other
  records of the position.
- No amounts of money anywhere in this requirement: time is counted in minutes, displayed to the minute.

### Open questions

None.

### Change log

- 2026-10-19 — created
- 2026-10-19 — added AC-DECIDE-16: employee edits a record awaiting decision — PM's own decision
- 2026-10-19 — added AC-DECIDE-17: employee cancels a returned record — PM's own decision
- 2026-10-19 — AC-DECIDE-15: "anyone with access" replaced with "the employee or any responsible manager of the position" — PM's own decision
- 2026-10-19 — open question answered: the 5 statuses are final — moved to Assumptions — PM's own decision
- 2026-10-19 — open question answered: a manager's own record is decided only by an administrator — added AC-DECIDE-18, AC-DECIDE-19 — PM's own decision
- 2026-10-19 — open question answered: records of a position with no manager are handled by the administrator — moved to Assumptions — PM's own decision
- 2026-10-19 — Assumptions: added that the requirement involves no money amounts — PM's own decision
- 2026-10-19 — all ACs: keywords temporarily switched to the template's Ukrainian form, then back to Given/When/Then; meaning and IDs unchanged — PM's own decision
- 2026-10-19 — Checked against the checklist: items 1–20 yes. Fixed: added AC-DECIDE-16, -17 (employee edits and cancels), AC-DECIDE-18, -19 (a manager's own record is decided by an administrator); clarified in AC-DECIDE-15 who sees the history; added "no money amounts" to Assumptions. Kept on purpose: 19 ACs, more than the 6–10 guideline. AI chat: no.
- 2026-10-19 — document fully in English, section headings and change-log entries included — PM's own decision
- 2026-10-20 — added AC-DECIDE-20: the manager sees the employee's optional comment (Main scenario step 2 now matches requirement 01) — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/1
- 2026-10-20 — added AC-DECIDE-21, AC-DECIDE-22 and assumptions: the employee changes only hours; the edited record is not checked against itself — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/2
