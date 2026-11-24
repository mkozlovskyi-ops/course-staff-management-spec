# Requirement: an employee chooses a substitute for each record, and an assistant who replaces a teacher is marked with an upgrade

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Today, who covers for an absent employee is agreed verbally, and the manager learns about it only on the day. When an
assistant replaces a teacher, payroll has to find out separately that those hours are paid at a higher rate. With this
requirement the manager keeps a list of possible substitutes for each employee, the employee picks one of them for
each record, the portal does not allow a substitute who is busy at that time, and a record where an assistant replaces
a teacher is marked "With upgrade" with the minutes, so payroll does not need to ask.

### Users and roles

- Responsible manager (`manager`) — keeps the substitute list of each employee of their positions; sees the substitute
  and the upgrade mark on records of their positions.
- Administrator (`admin`) — can keep the substitute list of any employee.
- Employee (`employee`) — chooses a substitute from their own list when requesting leave (optional); sees their own
  substitutions — the records where they cover for someone.
- A person who is not signed in sees nothing.

### Main scenario

1. The manager adds employees to the substitute list of an employee of their positions.
2. When requesting leave, the employee may choose a substitute for each record from their own list; the portal does not
   accept a substitute who is busy at that time.
3. The record shows the substitute (or "No substitute"); the manager sees it when deciding.
4. If the substitute is an assistant and the absent employee's position is a teacher's, the record is marked
   "With upgrade" with the minutes of the substitution.
5. The substitute sees the record in "My substitutions".

### Acceptance criteria

- AC-SUBST-01: Given manager1@example.test is the manager of teacher1@example.test's main position and teacher1@example.test's substitute list is empty. When manager1@example.test adds assistant1@example.test and teacher2@example.test to teacher1@example.test's substitute list. Then the substitute list of teacher1@example.test shows exactly assistant1@example.test and teacher2@example.test.
- AC-SUBST-02: Given teacher1@example.test's substitute list from AC-SUBST-01 and a main position with the schedule Mon–Fri 08:30–15:30. When teacher1@example.test requests "Personal leave" for the whole day Wednesday 2026-12-02, chooses the substitute assistant1@example.test and confirms the summary. Then 1 record 2026-12-02 08:30–15:30 is created with status "Awaiting decision" and the substitute assistant1@example.test, and manager1@example.test sees the substitute on that record in "Awaiting decision".
- AC-SUBST-03: Given teacher1@example.test's substitute list from AC-SUBST-01. When teacher1@example.test requests the whole day 2026-12-02 without choosing a substitute and confirms the summary. Then the record is created, and both teacher1@example.test and manager1@example.test see "No substitute" on it.
- AC-SUBST-04: Given teacher1@example.test's substitute list from AC-SUBST-01 and the employee teacher3@example.test who is not on it. When teacher1@example.test chooses a substitute for a record. Then the choice offers exactly assistant1@example.test and teacher2@example.test, and teacher3@example.test is not offered.
- AC-SUBST-05: Given assistant1@example.test has an own record 2026-12-02 09:00–11:00 with status "Awaiting decision". When teacher1@example.test chooses assistant1@example.test as the substitute for the record 2026-12-02 08:30–15:30 and confirms. Then no request is created, and teacher1@example.test sees the message "assistant1@example.test is not available at this time: 2026-12-02 09:00–11:00".
- AC-SUBST-06: Given assistant1@example.test is the substitute in teacher2@example.test's record 2026-12-02 10:00–12:00 with status "Approved". When teacher1@example.test chooses assistant1@example.test as the substitute for the record 2026-12-02 08:30–15:30 and confirms. Then no request is created, and teacher1@example.test sees the message "assistant1@example.test is not available at this time: 2026-12-02 10:00–12:00".
- AC-SUBST-07: Given assistant1@example.test's only record on 2026-12-02 has status "Rejected". When teacher1@example.test chooses assistant1@example.test as the substitute for the record 2026-12-02 08:30–15:30 and confirms. Then the record is created with the substitute assistant1@example.test.
- AC-SUBST-08: Given teacher1@example.test's position is "Elementary school / Teacher" and assistant1@example.test's main position is "Elementary school / Assistant". When teacher1@example.test requests 2026-12-02 from 08:30 to 12:00 with the substitute assistant1@example.test and confirms. Then the record shows the substitute assistant1@example.test and the mark "With upgrade: 210 min".
- AC-SUBST-09: Given teacher2@example.test's main position is "High school / Math teacher". When teacher1@example.test requests 2026-12-02 from 08:30 to 12:00 with the substitute teacher2@example.test and confirms. Then the record shows the substitute teacher2@example.test and has no "With upgrade" mark.
- AC-SUBST-10: Given the record from AC-SUBST-02. When assistant1@example.test opens "My substitutions". Then assistant1@example.test sees 1 line: 2026-12-02 08:30–15:30, teacher1@example.test, status "Awaiting decision".
- AC-SUBST-11: Given teacher1@example.test's substitute list is empty. When teacher1@example.test requests the whole day 2026-12-02. Then instead of a choice of substitutes teacher1@example.test sees the message "Your manager has not added substitutes for you yet", and the request can still be confirmed with "No substitute".
- AC-SUBST-12: Given teacher1@example.test is signed in and has only the employee role. When teacher1@example.test opens teacher2@example.test's substitute list for changing by a direct link. Then teacher1@example.test sees the message "Only the manager of this employee's position can change the substitute list", and teacher2@example.test's substitute list is unchanged.
- AC-SUBST-13: Given manager2@example.test is not the manager of any of teacher1@example.test's positions. When manager2@example.test tries to add teacher3@example.test to teacher1@example.test's substitute list directly, bypassing the portal's pages. Then teacher1@example.test's substitute list is unchanged and still shows exactly assistant1@example.test and teacher2@example.test.
- AC-SUBST-14: Given nobody is signed in. When a visitor opens "My substitutions" by a direct link. Then the visitor sees the sign-in page, and no records or names are shown.
- AC-SUBST-15: Given assistant1@example.test is not a substitute in any record with status "Awaiting decision", "Returned for revision" or "Approved". When assistant1@example.test opens "My substitutions". Then assistant1@example.test sees the message "You are not covering for anyone".

### Out of scope

- The employee changing the substitute of a submitted record: before a decision only the manager changes it
  (`08-on-behalf-and-edit.md`); the employee can still change only the hours (AC-DECIDE-21).
- The substitute agreeing to or declining a substitution in the portal: the employee arranges it in person.
- Checking the substitute's own schedule (whether they work at that time): only their leave and other substitutions
  are checked.
- Amounts of money for the upgrade: the portal shows only the mark and the minutes; payroll calculates the pay.
- Upgrades for other combinations of positions (for example, office staff replacing a teacher).
- A substitute for part of a record: one substitute covers the whole record.

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv).
- Employees, positions and managers for testing are created by the developer as test data (or by the administrator,
  `07-staff-management.md`); `assistant1@example.test` has the main position "Elementary school / Assistant".
- The substitute list belongs to an employee, not to a position; any responsible manager of any of the employee's
  positions, or an administrator, can add to it or remove from it. An employee cannot be on their own list.
- The substitute is chosen per record; a request with several records may have different substitutes or none.
- A substitute is busy (AC-SUBST-05, -06) when, at any minute of the record, they have an own record or are the
  substitute in another record with status "Awaiting decision", "Returned for revision" or "Approved". "Rejected" and
  "Cancelled" records do not make them busy (AC-SUBST-07). If several records make them busy, the message names the
  first one by time.
- The upgrade mark is set automatically: the absent employee's position on that record has the job title "Teacher"
  (any job title that contains "teacher", for example "Math teacher") and the substitute's main position has the job
  title "Assistant". The minutes are the record's minutes inside the schedule, the same as its duration.
- Removing someone from the substitute list does not change existing records: they keep their substitute.
- A deactivated employee (`07-staff-management.md`) is not offered as a substitute.
- No amounts of money anywhere in this requirement: time is counted in minutes.

### Open questions

None.

### Change log

- 2026-11-24 — created; roadmap row WS8 (substitutes and the assistant upgrade) — PM's own decisions: substitute optional, only from the manager's list, a busy substitute is not accepted, the upgrade mark is automatic
- 2026-11-24 — Checked against the checklist: items 1–16 yes. Refusals of two kinds: forbidden role (AC-SUBST-12 employee by a direct link, AC-SUBST-13 manager of other positions bypassing the pages) and not signed in (AC-SUBST-14); interface state: empty substitute list (AC-SUBST-11) and no substitutions (AC-SUBST-15). New terms "Substitute list" and "Upgrade" added to the glossary. Kept on purpose: 15 ACs, more than the 8–12 guideline. AI chat: no.
