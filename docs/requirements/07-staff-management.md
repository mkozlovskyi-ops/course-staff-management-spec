# Requirement: the administrator manages employees, positions, schedules, managers and deputies, and every user changes their own password

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Until now employees, positions, schedules and managers existed only as test data that developers created by hand, so the
school could not hire a new teacher, change a schedule or cover for a manager on leave without a developer. With this
requirement the administrator does all of this in the portal: creates employees and gives them a one-time sign-in link,
creates positions with schedules and responsible managers, assigns a deputy for a manager's absence and deactivates
people who leave — without losing the history of their leave.

### Users and roles

- Administrator (`admin`) — creates and changes employees (including names and emails), positions, schedules and
  managers; assigns deputies; deactivates employees. Cannot change their own role.
- Responsible manager (`manager`) — does not manage people; while assigned as a deputy, sees and decides the records of
  the absent manager's positions.
- Employee (`employee`) — sets a password through the administrator's link and later changes their own password; cannot
  change their own name or email.
- A person who is not signed in sees only the sign-in page and the page of a valid one-time link.

### Main scenario

1. The administrator creates an employee with a name and an email and gets a one-time sign-in link valid for 7 days.
2. The administrator gives the link to the person in person or through the school's usual channel; the person opens it
   and sets their password.
3. The administrator creates the employee's positions: type, division, job title, weekly schedule with the date it
   applies from, and responsible managers.
4. When a manager is absent, the administrator assigns a deputy for a date range.
5. When an employee leaves, the administrator deactivates them; their records and balances stay in the history.
6. Any signed-in user can change their own password.

### Acceptance criteria

- AC-STAFF-01: Given the administrator is signed in and there is no employee teacher3@example.test. When the administrator creates the employee "Teacher Three" with the email teacher3@example.test on 2026-12-01. Then the employee list shows "Teacher Three", teacher3@example.test, status "Invited", and the administrator sees a one-time sign-in link valid until 2026-12-08.
- AC-STAFF-02: Given the link from AC-STAFF-01 and today is 2026-12-03. When the person opens the link and sets a password of 10 characters. Then they are signed in as teacher3@example.test and see the message "Your password is set", and the employee list shows teacher3@example.test with status "Active".
- AC-STAFF-03: Given the link from AC-STAFF-01 and today is 2026-12-09. When the person opens the link. Then they see the message "This link has expired. Ask the administrator for a new one", and no password can be set.
- AC-STAFF-04: Given the active employee teacher3@example.test without positions. When the administrator creates for teacher3@example.test the main position "Elementary school / Teacher" with the schedule Mon–Fri 08:30–15:30 from 2026-12-07 and the responsible manager manager1@example.test. Then teacher3@example.test sees this position and its schedule in "Your weekly schedule", and manager1@example.test sees teacher3@example.test among the employees of their positions.
- AC-STAFF-05: Given teacher3@example.test has the main position from AC-STAFF-04. When the administrator tries to create another main position "High school / Math teacher" for teacher3@example.test. Then the position is not saved, and the administrator sees the message "An employee has exactly one main position".
- AC-STAFF-06: Given teacher3@example.test has the main position with the schedule Mon 08:30–15:30. When the administrator creates the additional position "High school / Math teacher" with the schedule Mon 15:00–17:00. Then the position is not saved, and the administrator sees the message "This schedule overlaps the position Elementary school / Teacher: Mon 08:30–15:30".
- AC-STAFF-07: Given teacher1@example.test's main position has the schedule Mon–Fri 08:30–15:30 and a record 2026-12-08 08:30–15:30 with status "Approved". When the administrator sets the new schedule Mon–Fri 08:30–12:30 from 2026-12-07 and teacher1@example.test then requests the whole day 2026-12-09. Then the record 2026-12-08 still shows 08:30–15:30, and the new record 2026-12-09 is 08:30–12:30.
- AC-STAFF-08: Given the active employee teacher3@example.test with a record 2026-12-08 with status "Approved". When the administrator deactivates teacher3@example.test and teacher3@example.test then tries to sign in. Then teacher3@example.test sees the message "Your account is deactivated. Contact the administrator" and is not signed in, and manager1@example.test still sees the record 2026-12-08 of teacher3@example.test in "History".
- AC-STAFF-09: Given teacher1@example.test has a record 2026-12-08 with status "Awaiting decision" of a position whose only manager is manager1@example.test. When the administrator assigns manager2@example.test as the deputy of manager1@example.test from 2026-12-07 to 2026-12-11 and manager2@example.test opens "Awaiting decision" on 2026-12-08. Then manager2@example.test sees the record 2026-12-08 of teacher1@example.test and can approve it, and manager1@example.test also still sees it.
- AC-STAFF-10: Given the deputy assignment from AC-STAFF-09 and a new record of teacher1@example.test 2026-12-15 with status "Awaiting decision". When manager2@example.test opens "Awaiting decision" on 2026-12-14. Then manager2@example.test does not see any record of teacher1@example.test.
- AC-STAFF-11: Given the employee teacher1@example.test named "Teacher One". When the administrator changes the name to "Teacher One-Smith" and the email to teacher1-new@example.test. Then the employee list shows "Teacher One-Smith", teacher1-new@example.test, and the person signs in with teacher1-new@example.test and the same password.
- AC-STAFF-12: Given teacher1@example.test is signed in and has only the employee role. When teacher1@example.test tries to change their own name or email. Then teacher1@example.test sees the message "Only an administrator can change names and emails", and both the name and the email stay the same.
- AC-STAFF-13: Given manager1@example.test is signed in and is not an administrator. When manager1@example.test opens the page for creating employees by a direct link. Then manager1@example.test sees the message "Only an administrator can manage employees", and the employee list is unchanged.
- AC-STAFF-14: Given manager1@example.test is not an administrator. When manager1@example.test tries to assign themselves as a deputy directly, bypassing the portal's pages. Then no deputy assignment exists, and manager1@example.test still sees only the records of their own positions.
- AC-STAFF-15: Given nobody is signed in. When a visitor opens the employee list by a direct link. Then the visitor sees the sign-in page, and no names or emails are shown.
- AC-STAFF-16: Given teacher1@example.test is signed in. When teacher1@example.test enters the correct current password and a new password of 12 characters. Then teacher1@example.test sees the message "Your password has been changed", and on the next sign-in only the new password works.
- AC-STAFF-17: Given teacher1@example.test is signed in. When teacher1@example.test enters a wrong current password and a new password. Then teacher1@example.test sees the message "The current password is incorrect", and the old password still works.
- AC-STAFF-18: Given the administrator is the only administrator. When the administrator tries to remove their own administrator role. Then the role is not changed, and the administrator sees the message "You cannot change your own role".
- AC-STAFF-19: Given no deputy has ever been assigned. When the administrator opens the list of deputies. Then the administrator sees the message "No deputies assigned".

### Out of scope

- Emails to new users: the portal never sends them (developers' mail limits); the administrator passes the one-time
  link themselves.
- Self-registration: only the administrator creates employees.
- Deleting employees: only deactivation, so history and balances stay.
- Reactivating a deactivated employee and resending an expired link may be done by the administrator, but the steps
  are the developer's choice; only the results in the ACs are required.
- Changing the number of leave days per school year (18) and carried-over days in settings.
- Importing employees from a spreadsheet.
- Password recovery by email ("forgot password"): the administrator creates a new one-time link.

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv).
- The first administrator is created by the developer as test data; after that administrators are granted only by
  another administrator.
- A one-time link works once and for 7 days including the day it was created (created 2026-12-01 → valid until
  2026-12-08 inclusive). A new link makes the previous one invalid.
- A password has at least 8 characters; other password rules are the developer's choice.
- Email addresses are unique across all employees, active and deactivated.
- A schedule applies from the date the administrator sets; the previous schedule applies before it. Existing records
  never change when a schedule changes (AC-STAFF-07). Schedules of one employee's positions never overlap at any date
  (AC-STAFF-06; see `03-leave-balance.md`).
- A deputy assignment covers all positions of the absent manager, includes both dates, and gives the deputy the same
  rights as that manager on those positions for those days — including never deciding their own records. Decisions made
  by the deputy stay after the period ends, and the history shows that the deputy decided.
- A deactivated employee cannot sign in, is not offered as a substitute, and their records with status "Awaiting
  decision" or "Returned for revision" stay as they are for the manager to decide. Their records and balances remain in
  history and in the register.
- Changing a name or an email does not change any record; the records show the current name and email.
- No amounts of money anywhere in this requirement.

### Open questions

None.

### Change log

- 2026-11-24 — created; roadmap row "the administrator manages employees … and every user changes their own password" — PM's own decisions: deactivation instead of deletion, deputy decides alongside the manager, a schedule applies from a date, first sign-in by a one-time link from the administrator
- 2026-11-24 — refusal AC for "only the administrator changes names and emails" added (AC-STAFF-12) — https://github.com/mkozlovskyi-ops/course-staff-management-spec/issues/19
- 2026-11-24 — Checked against the checklist: items 1–16 yes. Refusals of two kinds: forbidden role (AC-STAFF-12, -13, -14 bypassing the pages, -18) and not signed in (AC-STAFF-15); interface state: no deputies (AC-STAFF-19) and the expired link (AC-STAFF-03). New terms "One-time link" and "Deactivated employee" added to the glossary; "timetable" replaced with "schedule". Kept on purpose: 19 ACs — the requirement covers one intent (the administrator manages people) but many objects. AI chat: no.
