# Requirement: an approved record is cancelled through the manager, and its days return to the balance

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Plans change after leave is approved, but today an approved record cannot be undone: the days stay deducted from the
balance and the school calendar cannot close that day. With this requirement the employee asks to cancel an approved
record in the future, the manager confirms or declines with a reason, and a manager can also cancel a record with a
note; the days return to the balance at once.

### Users and roles

- Employee (`employee`) — asks to cancel their own approved records with a date today or later; sees the result.
- Responsible manager (`manager`) — confirms or declines cancellation requests for records of their positions; cancels
  an approved record of their positions with a note.
- Administrator (`admin`) — does the same for any record.
- A person who is not signed in sees nothing.

### Main scenario

1. The employee asks to cancel an approved record with a date today or later and gives a reason.
2. The record stays "Approved" and is marked "Cancellation requested"; the manager sees it in "Awaiting decision".
3. The manager confirms — the record becomes "Cancelled" and its days return to the balance — or declines with a
   comment, and the record stays "Approved".
4. A manager may also cancel an approved record of their positions directly, with a note to the employee.

### Acceptance criteria

- AC-CANCEL-01: Given today is 2026-12-01 and teacher1@example.test has a record 2026-12-10 "Personal leave" 08:30–15:30 with status "Approved". When teacher1@example.test asks to cancel it with the reason "Plans changed". Then teacher1@example.test sees the record with status "Approved" and the mark "Cancellation requested", and manager1@example.test sees it in "Awaiting decision" with "Cancellation requested: Plans changed".
- AC-CANCEL-02: Given the request from AC-CANCEL-01 and teacher1@example.test's "Personal leave" shows used 3.00, remaining 15.00. When manager1@example.test confirms the cancellation. Then the record has status "Cancelled", and teacher1@example.test's "Personal leave" shows used 2.00, remaining 16.00.
- AC-CANCEL-03: Given the request from AC-CANCEL-01. When manager1@example.test declines it with the comment "Exam week". Then the record keeps status "Approved" without the mark "Cancellation requested", and teacher1@example.test sees "Cancellation declined: Exam week" on it.
- AC-CANCEL-04: Given the request from AC-CANCEL-01. When manager1@example.test declines it without a comment. Then the record still shows "Cancellation requested", and manager1@example.test sees the message "Tell the employee why the cancellation is declined".
- AC-CANCEL-05: Given today is 2026-12-01 and teacher1@example.test has a record 2026-11-27 with status "Approved". When teacher1@example.test tries to ask to cancel it. Then no cancellation is requested, the record keeps status "Approved", and teacher1@example.test sees the message "Records in the past cannot be cancelled".
- AC-CANCEL-06: Given the record from AC-CANCEL-01 already has "Cancellation requested". When teacher1@example.test asks to cancel it again. Then teacher1@example.test sees the message "Cancellation has already been requested", and there is still one request.
- AC-CANCEL-07: Given a whole-day "Personal leave" record approved with the split "0.50 Personal leave + 0.50 Unpaid leave" (as in AC-BALANCE-10), with a date in the future. When the manager confirms its cancellation. Then "Personal leave" used decreases by 0.50 and "Unpaid leave" used decreases by 0.50.
- AC-CANCEL-08: Given today is 2026-12-01 and teacher1@example.test has a record 2026-12-10 with status "Approved". When manager1@example.test cancels it with the note "Event moved to January". Then the record has status "Cancelled", and teacher1@example.test sees the note "Event moved to January" on it in "My requests".
- AC-CANCEL-09: Given the record from AC-CANCEL-08 with status "Approved". When manager1@example.test tries to cancel it without a note. Then the record keeps status "Approved", and manager1@example.test sees the message "Tell the employee why the record is cancelled".
- AC-CANCEL-10: Given teacher1@example.test has a record 2026-12-10 with status "Approved". When teacher2@example.test opens the cancellation of that record by a direct link. Then teacher2@example.test sees the message "You can cancel only your own records", and the record keeps status "Approved" without a cancellation request.
- AC-CANCEL-11: Given the request from AC-CANCEL-01, and manager2@example.test is not a manager of that record's position. When manager2@example.test tries to confirm the cancellation directly, bypassing the portal's pages. Then the record keeps status "Approved" with "Cancellation requested", and teacher1@example.test's balances are unchanged.
- AC-CANCEL-12: Given nobody is signed in. When a visitor opens the cancellation of a record by a direct link. Then the visitor sees the sign-in page, and no record is shown.
- AC-CANCEL-13: Given the record 2026-12-10 was cancelled in AC-CANCEL-02 and there are no other records on 2026-12-10. When the administrator adds the closure "Staff day" for 2026-12-10. Then the closure is saved, and the school calendar shows "Staff day" on 2026-12-10.

### Out of scope

- Cancelling records in the past (AC-CANCEL-05): such a correction is made by the administrator outside this slice.
- Cancelling part of a record (some hours): the employee asks to cancel the record and submits a new one.
- Withdrawing a cancellation request by the employee.
- Emails about the result: only the status, the mark and the comment in "My requests".

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv). "Today or later" is compared with today's date in Kyiv.
- A cancellation request is decided by any responsible manager of the record's position, a deputy for that day, or an
  administrator; a manager never decides a cancellation request for their own record — an administrator does, as with
  decisions (AC-DECIDE-18).
- A confirmed cancellation returns the days to the balance of the school year of the record's date, including both
  parts of a split record (AC-CANCEL-07). A declined request changes nothing.
- A record with "Cancellation requested" still counts as "Approved": in the balance, for overlaps, for the
  substitute's availability and for closing a day (AC-CAL-07).
- After a cancellation the substitute is free at that time, and the date can be closed in the school calendar
  (AC-CAL-08, AC-CANCEL-13).
- The history of the record shows the request, the decision and who made them, to the minute.
- No amounts of money anywhere in this requirement.

### Open questions

None.

### Change log

- 2026-11-24 — created; part of roadmap row WS11 — PM's own decisions: the employee requests, the manager confirms or declines with a comment; records in the past are not cancelled; a manager may cancel with a note
- 2026-11-24 — Checked against the checklist: items 1–16 yes. Refusals of two kinds: other people's data (AC-CANCEL-10), forbidden role bypassing the pages (AC-CANCEL-11) and not signed in (AC-CANCEL-12); interface state: "Cancellation requested" mark (AC-CANCEL-01) and the errors with exact text (AC-CANCEL-04, -05, -06, -09). New term "Cancellation request" added to the glossary. AI chat: no.
