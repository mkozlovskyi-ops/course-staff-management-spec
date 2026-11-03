# Requirement: the administrator closes days and hours in the school calendar, and they cannot be taken as leave

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Today holidays, school breaks and early dismissals are known only from announcements, so employees request leave for
days when the school is closed anyway, and those days are deducted from their balance. With this requirement the
administrator records closures in one school calendar that everyone sees; a closed day or closed hours cannot be
chosen in a leave request and do not count as working time in balances.

### Users and roles

- Administrator (`admin`) — adds closures to the school calendar and removes them. Only the administrator can change
  the calendar.
- Employee (`employee`) — sees the school calendar; cannot choose closed days or closed hours in a leave request.
- Responsible manager (`manager`) — sees the school calendar like an employee; cannot change it.
- A person who is not signed in sees nothing of the calendar.

### Main scenario

1. The administrator adds a closure to the school calendar: a name and either whole days (a start date and an end
   date, both included) or an early dismissal (one date and the time from which the school is closed).
2. The portal saves the closure if none of its days or hours has leave records that block it, and shows it in the
   school calendar.
3. Every signed-in user sees the closure in the school calendar: name and dates (and the time for an early
   dismissal).
4. When an employee requests leave, closed days are not accepted and closed hours are not part of their schedule on
   that date.
5. Balances count a day's share only from the working minutes that are not closed.
6. If the administrator made a mistake, they remove the closure, and the days become working days again according to
   the schedules.

### Acceptance criteria

- AC-CAL-01: Given the school calendar has no closures. When the administrator adds the closure "Winter break" for whole days from 2026-12-28 to 2027-01-08. Then the school calendar shows "Winter break" with the dates 2026-12-28 – 2027-01-08.
- AC-CAL-02: Given the administrator added "Winter break" 2026-12-28 – 2027-01-08. When teacher1@example.test opens the school calendar. Then teacher1@example.test sees "Winter break" with the dates 2026-12-28 – 2027-01-08.
- AC-CAL-03: Given "Winter break" 2026-12-28 – 2027-01-08 and an employee with the schedule Mon–Fri 08:30–15:30. When the employee tries to choose Tuesday 2026-12-29 in a new request. Then the date is not accepted, and the employee sees the message "The school is closed on this day: Winter break".
- AC-CAL-04: Given an early dismissal "Teachers' meeting" on Friday 2026-11-20 from 12:00 and an employee with the schedule Mon–Fri 08:30–15:30. When the employee requests "Personal leave" for the whole day 2026-11-20 and confirms the summary. Then 1 record 2026-11-20 08:30–12:00 is created.
- AC-CAL-05: Given the early dismissal from AC-CAL-04 and the same employee. When the employee enters the hours 13:00–14:00 on 2026-11-20. Then no request is created, and the employee sees the message "Hours must be within your schedule: 08:30–12:00".
- AC-CAL-06: Given the early dismissal from AC-CAL-04, an employee with a single position (schedule Fri 08:30–15:30) and "Sick leave" used 0.00. When the manager approves the whole-day "Sick leave" record 2026-11-20 08:30–12:00. Then the employee's balances show "Sick leave" used 1.00 (210 of 210 working minutes that day), not 0.50.
- AC-CAL-07: Given teacher1@example.test has a record 2026-12-29 08:30–15:30 with status "Approved". When the administrator tries to add "Winter break" from 2026-12-28 to 2027-01-08. Then the closure is not saved, the school calendar has no "Winter break", and the administrator sees the message "These days have leave records and cannot be closed: 2026-12-29 teacher1@example.test".
- AC-CAL-08: Given the only record on 2026-12-29 has status "Rejected". When the administrator adds "Winter break" from 2026-12-28 to 2027-01-08. Then the closure is saved, and the school calendar shows "Winter break" with the dates 2026-12-28 – 2027-01-08.
- AC-CAL-09: Given "Winter break" 2026-12-28 – 2027-01-08 and an employee with the schedule Mon–Fri 08:30–15:30. When the administrator removes "Winter break" and the employee then chooses Tuesday 2026-12-29 in a new request. Then the school calendar no longer shows "Winter break", and 2026-12-29 is accepted with the hours 08:30–15:30.
- AC-CAL-10: Given the administrator adds a closure. When the administrator enters the start date 2027-01-08 and the end date 2026-12-28. Then the closure is not saved, and the administrator sees the message "The end date cannot be earlier than the start date".
- AC-CAL-11: Given "Winter break" 2026-12-28 – 2027-01-08. When the administrator tries to add the closure "Staff day" for 2026-12-30. Then the closure is not saved, and the administrator sees the message "2026-12-30 is already closed: Winter break".
- AC-CAL-12: Given teacher1@example.test is signed in and has only the employee role. When teacher1@example.test opens the page for changing the school calendar by a direct link. Then teacher1@example.test sees the message "Only an administrator can change the school calendar", and the school calendar is unchanged.
- AC-CAL-13: Given manager1@example.test is signed in and is not an administrator. When manager1@example.test tries to close 2026-11-27 directly, bypassing the portal's pages. Then the school calendar is unchanged, and 2026-11-27 is still accepted in a new request.
- AC-CAL-14: Given nobody is signed in. When a visitor opens the school calendar by a direct link. Then the visitor sees the sign-in page, and no closure names or dates are shown.
- AC-CAL-15: Given the school calendar has no closures in school year 2026–2027. When an employee opens the school calendar for school year 2026–2027. Then the employee sees the message "No closed days in school year 2026–2027".

### Out of scope

- A closure for one division only (for example, an early dismissal in the elementary school only): every closure
  applies to the whole school.
- Changing a saved closure (its name, dates or time): the administrator removes it and adds a new one.
- Cancelling or moving existing leave records when a day is closed: a day with blocking records cannot be closed
  (AC-CAL-07); cancelling an approved record is slice WS11.
- A closure that starts in the morning and ends before the end of the day (for example, a late start): only whole
  days and early dismissals.
- Repeating closures (for example, the same holiday every year): each closure is added for specific dates.
- Notifying employees about a new closure: they see it in the school calendar; no emails.
- Weekly days off (weekends): they come from the schedules, not from the school calendar.

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv), the same for all users.
- Closures are created by the administrator in the portal; for testing, the developer creates the administrator
  account and any starting closures as test data.
- A closure has a name of up to 100 characters (for example, "Winter break", "Teachers' meeting").
- Whole-day closures include both the start date and the end date. A closure may be a single date (the start date
  equals the end date).
- An early dismissal closes one date from the given time to the end of the day; the time is chosen with 5-minute
  precision. Schedule blocks that start at or after that time are closed; a block that is running at that time ends
  at that time.
- A closed day is not a working day (see the glossary): it is not accepted in a new request, as in AC-CAL-03. In a
  multi-date request, a closed date is dropped with its message and the other dates stay, as with a date without
  working hours (AC-LEAVE-21).
- "Whole day" on an early dismissal date means the schedule blocks before the closing time (AC-CAL-04).
- Balances count a day's share as the record's minutes ÷ the employee's working minutes on that date that are not
  closed (AC-CAL-06); the rest of the balance rules are as in `03-leave-balance.md`.
- Records with status "Awaiting decision", "Returned for revision" or "Approved" block closing their date; records
  with status "Rejected" or "Cancelled" do not (AC-CAL-07, AC-CAL-08). For an early dismissal, only records with
  minutes after the closing time block it.
- Until slice WS11 an approved record cannot be cancelled, so a date with an approved record cannot be closed in
  this slice.
- When several records block a closure, the message lists all of them, one "date employee-email" pair per record,
  separated by ", ", in date order.
- Two closures may not share a date (AC-CAL-11).
- The school calendar shows closures of one school year (1 September – 31 August) at a time, in date order; a
  whole-day closure that crosses 31 August is shown in both school years.
- Removing a closure does not change any leave record: it only makes the days working days again.
- Closures can be added and removed for past dates as well; the same blocking rules apply.
- No amounts of money anywhere in this requirement.

### Open questions

- Question #20 (an approved record on a day being closed) is answered by AC-CAL-07: the day cannot be closed —
  https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/20

### Change log

- 2026-11-03 — created; slice WS7 in the roadmap, implemented by the pipeline in WS9 — PM's own decision
- 2026-11-03 — a day with blocking leave records cannot be closed (AC-CAL-07, AC-CAL-08) — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/20
- 2026-11-03 — Checked against the checklist: items 1–16 yes. Refusals of two kinds: forbidden role (AC-CAL-12, AC-CAL-13 — direct attempt bypassing the pages) and not signed in (AC-CAL-14); interface state: empty calendar (AC-CAL-15) and error messages (AC-CAL-03, -05, -07, -10, -11). New terms "School calendar", "Closure", "Early dismissal" added to the glossary. Kept on purpose: 15 ACs, more than the 8–12 guideline; AC-CAL-09 checks one behaviour (a removed closure makes the day a working day again) from two sides. AI chat: no.
