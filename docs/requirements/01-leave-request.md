# Requirement: an employee requests leave according to their schedule

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Today an employee asks for leave verbally or in a spreadsheet, and the manager cannot see exactly which hours and
positions the person will be absent from. An employee with two positions has to arrange it with two managers
separately. With this requirement the employee chooses dates, positions and hours within their schedule in one request,
and each position gets its own record that goes to that position's manager.

### Users and roles

- Employee (`employee`) — submits requests only on their own behalf and sees only their own requests and records.
- Responsible manager (`manager`) — does nothing in this requirement; receives the records the employee creates via
  requirement `02-leave-decision.md`.
- Administrator (`admin`) — does not act in this requirement; employees, positions and schedules for testing are
  created in advance (see "Assumptions").

### Main scenario

1. The employee chooses one or more dates they want leave for.
2. For each date the portal shows the employee's positions that have a schedule on that weekday, with each position's
   working hours.
3. For each position on each date the employee decides whether to include it; if included — whole day or specific
   hours within the schedule.
4. The employee chooses the leave type (for "Family celebration" — also the relationship of the person the celebration
   is for) and may add an optional comment.
5. The employee reviews a summary — dates, hours, positions, leave type and comment — and confirms.
6. The portal creates one leave record per included position per date, each with status "Awaiting decision".
7. The employee sees their records in the "My requests" list with the status of each.

### Acceptance criteria

- AC-LEAVE-01: Given an employee with main position "Elementary school / Teacher", schedule Mon–Fri 08:30–15:30. When the employee requests "Sick leave" for the whole day on Monday 2026-11-02 and confirms the summary. Then 1 leave record is created: position "Elementary school / Teacher", 2026-11-02, 08:30–15:30, type "Sick leave", status "Awaiting decision".
- AC-LEAVE-02: Given an employee with main position "Elementary school / Teacher" (Mon 08:30–15:30) and additional position "High school / Math teacher" (Mon 15:45–17:00). When the employee requests the whole day 2026-11-02 with both positions included. Then 2 records are created in one request: 08:30–15:30 for the main position and 15:45–17:00 for the additional position, both with status "Awaiting decision".
- AC-LEAVE-03: Given the employee with two positions from AC-LEAVE-02. When the employee requests 2026-11-02 and excludes the additional position. Then only 1 record is created — for the main position — and there is no record for the additional position.
- AC-LEAVE-04: Given an employee with a main position, schedule Mon–Fri 08:30–15:30. When the employee requests "Personal leave" on 2026-11-02 from 09:00 to 11:30. Then 1 record 2026-11-02 09:00–11:30 is created with a duration of 2 h 30 min.
- AC-LEAVE-05: Given an employee with a main position, schedule Mon–Fri 08:30–15:30. When the employee requests the dates 2026-11-02 and 2026-11-03, whole day. Then 2 records are created (one per date), both belonging to the same request.
- AC-LEAVE-06: Given an employee whose position has no Saturday in its schedule. When the employee tries to choose Saturday 2026-11-07. Then the date is not accepted, and the employee sees the message "You have no working hours on this day".
- AC-LEAVE-07: Given a position with the schedule block 08:30–13:00. When the employee enters the hours 08:00–10:00. Then no request is created, and the employee sees the message "Hours must be within your schedule: 08:30–13:00".
- AC-LEAVE-08: Given a position with the schedule block 08:30–13:00. When the employee enters 13:00 as the start. Then no request is created, and the employee sees the message "13:00 is the end of working time and can only be chosen as the end of leave".
- AC-LEAVE-09: Given a position with the schedule 08:30–15:30. When the employee enters the hours from 11:00 to 10:00. Then no request is created, and the employee sees the message "The end must be later than the start".
- AC-LEAVE-10: Given the employee chose the date 2026-11-02 and excluded all positions. When the employee tries to submit the request. Then no request is created, and the employee sees the message "Choose at least one position".
- AC-LEAVE-11: Given the employee filled in a request for 2 dates and sees the summary. When the employee does not confirm the summary and goes back to editing. Then no record is created, and the entered dates and hours are kept for editing.
- AC-LEAVE-12: Given the employee chose the type "Family celebration". When the employee submits the request without stating the relationship. Then no request is created, and the employee sees the message "State your relationship to this person: sibling, child, parent, grandchild or grandparent. Not applicable to cousins, friends, neighbours or siblings-in-law".
- AC-LEAVE-13: Given the employee already has a main-position record 2026-11-02 09:00–11:00 with status "Awaiting decision". When the employee requests the same position on 2026-11-02 10:00–12:00. Then no request is created, and the employee sees the message "You already have a request for this time: 2026-11-02 09:00–11:00".
- AC-LEAVE-14: Given employee teacher1@example.test has 2 records and employee teacher2@example.test has 3 records. When teacher1@example.test opens "My requests". Then the list contains exactly 2 records — only their own — each with date, hours, position, leave type and status.
- AC-LEAVE-15: Given a position with the schedule blocks Tue 08:30–12:00 and 12:30–15:30. When the employee requests "Personal leave" on 2026-11-03 from 11:00 to 13:00. Then 1 record 2026-11-03 11:00–13:00 is created with a duration of 1 h 30 min (the gap 12:00–12:30 is not counted).
- AC-LEAVE-16: Given the employee already has a main-position record 2026-11-03 09:00–11:00 with status "Awaiting decision". When the employee confirms a whole-day request for 2026-11-02 and 2026-11-03. Then no record is created for either date, and the employee sees the message "You already have a request for this time: 2026-11-03 09:00–11:00".
- AC-LEAVE-17: Given the employee requests the whole day 2026-11-02 for both positions from AC-LEAVE-02 with the comment "Doctor's appointment". When the employee confirms the summary. Then both created records carry the comment "Doctor's appointment".
- AC-LEAVE-18: Given the employee enters a comment of 501 characters. When the employee tries to submit the request. Then no request is created, and the employee sees the message "The comment can be at most 500 characters".
- AC-LEAVE-19: Given a position with the schedule blocks Tue 08:30–12:00 and 12:30–15:30. When the employee enters the hours 07:30–09:00 on 2026-11-03. Then no request is created, and the employee sees the message "Hours must be within your schedule: 08:30–12:00, 12:30–15:30".
- AC-LEAVE-20: Given a position with the schedule blocks Tue 08:30–12:00 and 12:30–15:30. When the employee enters 12:00 as the start on 2026-11-03. Then no request is created, and the employee sees the message "12:00 is the end of working time and can only be chosen as the end of leave".
- AC-LEAVE-21: Given an employee with the schedule Mon–Fri 08:30–15:30. When the employee chooses Friday 2026-11-06 and Saturday 2026-11-07 in one whole-day request and confirms. Then Saturday is not accepted with the message "You have no working hours on this day", and 1 record is created, for 2026-11-06.

### Out of scope

- Holidays, school breaks and early dismissals in the school calendar — slice WS7; in this slice a working day is
  defined only by the position's schedule.
- Choosing a substitute and the assistant upgrade — slice WS8.
- A manager or administrator submitting a request on behalf of an employee — slice WS11.
- The reminder to talk to the administrator about Personal leave of 2 or more days — slice WS11.
- An employee cancelling an approved record — slice WS11.
- The warning about exceeding the balance — requirement `03-leave-balance.md`.

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv), the same for all users.
- Employees, their positions, divisions, schedules and responsible managers for testing are created by the developer
  as test data; managing them is slice WS10.
- Times are chosen with 5-minute precision; duration is counted in minutes.
- A schedule block includes its start time and excludes its end time: in the block 08:30–13:00 the latest possible
  leave start is 12:55.
- "Whole day" for a position means all blocks of its schedule on that date.
- A request may span a gap between schedule blocks; only the minutes inside the blocks count. The start and the end
  must each fall inside a block (the end may equal a block's end).
- A request is saved as a whole: either all of its records are created or none. A date without working hours is
  not accepted (AC-LEAVE-06) and the other dates of the request stay; "all or none" applies to the accepted dates.
- One leave type per request; different types mean different requests (confirmed by PM).
- A request may have one optional comment of up to 500 characters; it is stored on every record of the request.
- When hours are outside the schedule, the message lists all blocks of that day, separated by ", " in time order.
- Records of the same position may not overlap in time when they have status "Awaiting decision", "Returned for
  revision" or "Approved"; records with status "Rejected" or "Cancelled" do not block. A record that is being
  edited is not checked against itself.
- An employee may request a date in the past (for example, sick leave after the fact). There is no limit on how many
  days ahead or back a request can be made (confirmed by PM).
- No amounts of money anywhere in this requirement: time is counted in minutes, displayed to the minute.

### Open questions

None.

### Change log

- 2026-10-19 — created
- 2026-10-19 — added AC-LEAVE-15: a request may span a gap between schedule blocks, only scheduled minutes count — PM's own decision
- 2026-10-19 — added AC-LEAVE-16 and the assumption "a request is saved as a whole": all records or none — PM's own decision
- 2026-10-19 — open question answered: no limit on how many days ahead or back a request can be made — moved to Assumptions — PM's own decision
- 2026-10-19 — open question answered: one leave type per request — moved to Assumptions — PM's own decision
- 2026-10-19 — Assumptions: added that the requirement involves no money amounts — PM's own decision
- 2026-10-19 — all ACs: keywords temporarily switched to the template's Ukrainian form, then back to Given/When/Then; meaning and IDs unchanged — PM's own decision
- 2026-10-19 — Checked against the checklist: items 1–20 yes. Fixed: added AC-LEAVE-15 (a request spanning a gap in the schedule) and AC-LEAVE-16 (all records or none); added "no money amounts" to Assumptions. Kept on purpose: 16 ACs, more than the 6–10 guideline; AC-LEAVE-11 and AC-LEAVE-14 check one result with several fields. AI chat: no.
- 2026-10-19 — document fully in English, section headings and change-log entries included — PM's own decision
- 2026-10-20 — added AC-LEAVE-17, AC-LEAVE-18 and the assumption on an optional comment up to 500 characters; Main scenario steps 4–5 mention the comment — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/1
- 2026-10-20 — assumption on overlaps: a record being edited is not checked against itself — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/2
- 2026-10-20 — added AC-LEAVE-19, AC-LEAVE-20 and the assumption that the schedule message lists all blocks of the day — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/5
- 2026-10-21 — added AC-LEAVE-21 and the assumption: a date without working hours is dropped from a multi-date request with the message, the other dates stay — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/9
