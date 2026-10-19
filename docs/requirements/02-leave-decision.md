# Вимога: a manager decides on leave records of their positions

<!--
Do not rename the ### section headings: they match the "Вимога" form and /requirements-sync relies on them.
The repo is public: no real client names, people's names or their data.
-->

### Навіщо

Today a manager learns about an employee's absence from a conversation or a spreadsheet and does not always know
whether it concerns their division. When an employee has two positions with different managers, it is unclear who
approves. With this requirement each manager sees only the records of their own positions, decides on each record
separately, and the employee sees the result and the comment.

### Користувачі й ролі

- Responsible manager (`manager`) — sees and decides only records of the positions they are assigned to. Does not see
  records of other positions, even of the same employee.
- Administrator (`admin`) — sees and can decide any record.
- Employee (`employee`) — sees the decision and comment on their own records; corrects a returned record and resubmits
  it. Makes no decisions.

### Основний сценарій

1. The manager opens the "Awaiting decision" list — records of the positions they are responsible for.
2. The manager reviews a record: employee, position, date, hours, leave type, employee's comment.
3. The manager makes one of the decisions: approve, reject, or return for revision with a comment.
4. The record leaves the "Awaiting decision" list and appears in the manager's "History".
5. The employee sees the new record status and the manager's comment in "My requests".
6. If the record was returned, the employee corrects it and resubmits — the record is again in the "Awaiting
   decision" list of the same managers.

### Критерії приймання

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

### Поза межами

- A manager changing the date, hours, leave type or substitute before deciding — a later slice (WS11).
- Cancelling an approved record (by the employee through the manager, or by the manager with a note) — slice WS11.
- A deputy manager while the manager is absent — slice WS10.
- The warning to the manager about exceeding the balance — requirement `03-leave-balance.md`.
- The warning about a conflict with the school calendar — slice WS7.
- Emails to the manager or the employee: only the status and an in-app marker.
- Approving all records of one request in a single action.

### Припущення

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

### Відкриті питання

None.

### Журнал змін

- 2026-10-19 — created
- 2026-10-19 — added AC-DECIDE-16: employee edits a record awaiting decision (spec-workflow review, G3)
- 2026-10-19 — added AC-DECIDE-17: employee cancels a returned record (spec-workflow review, G4)
- 2026-10-19 — AC-DECIDE-15: "anyone with access" replaced with "the employee or any responsible manager of the position" (spec-workflow review, G7)
- 2026-10-19 — open question answered: the 5 statuses are final — moved to Assumptions
- 2026-10-19 — open question answered: a manager's own record is decided only by an administrator — added AC-DECIDE-18, AC-DECIDE-19
- 2026-10-19 — open question answered: records of a position with no manager are handled by the administrator — moved to Assumptions
