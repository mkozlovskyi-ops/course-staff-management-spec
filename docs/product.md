# Product: school staff leave portal

<!--
docs/ variant. Do not rename the ### section headings: developers compare versions by them. The repo is public: no real client names, people's names, addresses or amounts.
-->

### Vision

At the school, staff (teachers, assistants, office staff) ask for leave or for a few hours off verbally or through
spreadsheets. One employee often holds two positions in different divisions (for example, main position — elementary
school teacher, additional position — high school math teacher), and each position is approved by its own manager. As a
result, requests get lost, leave balances are counted by hand, and payroll receives an incomplete list of approved
absences.

The portal lets an employee request leave for a whole day or for specific hours according to their schedule, lets a
manager decide only on the positions they are responsible for, and lets the school count balances precisely and keep a
register of approved leave.

Success indicators:

- An employee requests leave for several dates and both of their positions in one go, without asking a manager
  verbally.
- Each manager sees only the records of their own positions in the "Awaiting decision" list and decides on each record
  separately.
- Sick leave and Personal leave balances are calculated automatically to 0.01 day and match the approved records.
- The register of approved leave is produced without manual copying, separately for each division.

### Users and roles

Users are created by the administrator: no invitation emails are sent. One person can hold several roles at once: a
manager or an administrator is also an employee and submits their own requests.

| Role (identifier) | Who | How the role is granted |
|---|---|---|
| `employee` | Teacher, assistant, office worker: submits own leave requests, sees own schedule and balances | every employee created by the administrator has this role |
| `manager` | Responsible manager of one or more positions (for example, the elementary school manager); decides on the records of those positions | only the administrator: assigns a person as manager of specific positions |
| `admin` | School administrator: manages employees, positions, schedules, managers and the school calendar; can decide on any record | only another administrator; cannot change their own role |

### Permission matrix

The "From slice" column says from which slice an action is available; before that slice no implementation offers it.

| Action | employee | manager | admin | From slice |
|---|---|---|---|---|
| See own schedule, requests and balances | yes | yes | yes | WS6 |
| Submit own leave request | yes | yes | yes | WS6 |
| Change the hours of own record with status "Awaiting decision" or "Returned for revision" | own only | own only | own only | WS6 |
| Cancel own record with status "Awaiting decision" or "Returned for revision" | own only | own only | own only | WS6 |
| Cancel own approved record (through the manager) | own only | own only | own only | WS11 |
| Submit a request on behalf of another employee | — | only employees of own positions | yes | WS11 |
| See other employees' leave records | — | only records of own positions | yes | WS6 |
| Approve, reject or return a record for revision | — | only records of own positions, never own records | yes | WS6 |
| Change another employee's record before deciding (date, hours, leave type, substitute) | — | only records of own positions | yes | WS11 |
| Cancel another employee's record | — | only records of own positions | yes | WS11 |
| See other employees' balances | — | only employees of own positions | yes | WS6 |
| See the school calendar | yes | yes | yes | WS7 |
| Change the school calendar (days off, holidays, early dismissals) | — | — | yes | WS7 |
| Create employees, positions, schedules; assign managers and deputies | — | — | yes | WS10 |
| Change own password | own only | own only | own only | WS10 |
| Change a user's name or email (including own) | — | — | yes | WS10 |
| Maintain an employee's substitute list | — | only employees of own positions | yes | WS8 |
| See the register of approved leave | — | only employees of own positions | yes | WS9 |
| See salaries or pay rates | — | — | — | never: not stored in the portal |

### Glossary

| Term | Definition | Do not use |
|---|---|---|
| Employee | A person who works at the school and has at least one position | staff member, user, personnel |
| Position | A paid position of an employee: type (main or additional), division, job title (teacher, assistant…), its own weekly schedule and one or more responsible managers | assignment, rate, slot, role (role means portal access only) |
| Main / additional position | The type of a position. An employee has exactly one main position and zero or more additional ones | primary, secondary, second job |
| Division | The part of the school a position belongs to (for example, elementary school, high school, early childhood) | department, faculty, school (school means the whole institution) |
| Schedule | The weekly working hours of a position: any number of time blocks per weekday, with gaps between blocks allowed | timetable, roster |
| Working day | A date on which the position's schedule has at least one block and which is not closed in the school calendar | business day, weekday |
| Leave request | One submission by an employee: one or more dates and one or more positions | application, ticket |
| Leave record | The part of a request for one position on one date with specific hours; decisions are made on records | row, slot, item |
| Leave type | Sick leave, Personal leave, Unpaid leave, Family celebration | leave kind, category |
| Record status | "Awaiting decision", "Approved", "Rejected", "Returned for revision", "Cancelled" | state, stage, pending |
| Balance | How many days of Sick leave or Personal leave the employee can still use in the school year, including carried-over days | limit, quota, allowance |
| School year | The period over which the balance is counted; unused days carry over to the next school year | calendar year, season |
| Responsible manager | A manager assigned by the administrator to a position; receives that position's records | supervisor, boss, administrator |
| Substitute | An employee who covers for the absent person during a record | deputy (a deputy is a manager, see below), cover |
| Deputy manager | A manager temporarily assigned by the administrator in place of an absent manager for a date range | substitute manager, backup |

### Slice roadmap

| Session | Slice (what the user will be able to do) | Requirements | Status |
|---|---|---|---|
| WS6 | An employee requests leave according to their schedule, a manager decides on each record, and the employee sees their balance | `01-leave-request.md`, `02-leave-decision.md`, `03-leave-balance.md` | in progress |
| WS7 | The administrator closes days and hours in the school calendar (holidays, early dismissals), and they cannot be taken as leave | — | planned |
| WS8 | An employee chooses a substitute for each record, and an assistant can temporarily replace a teacher with an upgrade | — | planned |
| WS9 | The administrator views and downloads the register of approved leave separately for each division | — | planned |
| WS10 | The administrator manages employees (including their names and emails), positions, schedules, managers and deputy managers, and every user changes their own password | — | planned |
| WS11 | A manager submits a request on behalf of an employee and edits a record's date, hours, leave type or substitute before deciding, and an employee gets a reminder about long Personal leave and cancels an approved record through the manager | — | planned |

### Out of scope

- Emails to users: developers' free mail plan allows only 2 emails per hour and only to members of their project.
  Reminders are an in-app marker; an "invitation" is a link from the administrator.
- Scheduled actions more often than once a day: developers' free hosting runs scheduled jobs at most once a day.
- Salaries, pay rates and payroll calculation: the portal neither stores nor shows them; payroll keeps them
  separately.
- Approving a record from an email or a message: decisions are made only in the portal.
- Mobile layout: the portal is designed for a computer.
- Migrating historical leave records from the previous system: the portal starts with clean data, and balances
  begin from the carried-over days the administrator enters.
- A voice assistant for submitting requests: extra setup effort and a per-minute usage cost; a possible later option.
- Verifying that the employee actually spoke with the administrator before long Personal leave: reminder only.

### Як доводимо

How each acceptance criterion of the delivered slices is proven. **e2e** — a browser scenario on production shows the
"Then" on screen (UAT recording); **database** — a test of the stored data proves that a rule holds even when someone
bypasses the portal's pages; **unit** — a test of a calculation or a boundary with exact numbers; **manual** — checked
by a person because it cannot be automated. A combination means both are needed: the database or unit test proves
the rule, the e2e scenario proves what the person sees.

Slice WS7 (school calendar) has no requirements yet, so the table covers slice WS6 only. No criterion needs "manual":
every "Then" of slice WS6 is either visible on screen or a rule in the data.

| AC ID | How we prove it | Why |
|---|---|---|
| AC-LEAVE-01 | e2e | The new record and all its fields are visible in "My requests" |
| AC-LEAVE-02 | e2e | Both records with their hours are visible in "My requests" |
| AC-LEAVE-03 | e2e | "My requests" shows one record and none for the excluded position |
| AC-LEAVE-04 | unit + e2e | Duration 2 h 30 min is a calculation (unit); the record with its duration is visible (e2e) |
| AC-LEAVE-05 | e2e | Two records of one request are visible in "My requests" |
| AC-LEAVE-06 | e2e | The exact message is shown on screen |
| AC-LEAVE-07 | unit + e2e | The schedule boundary is a rule with exact times (unit); the exact message is shown (e2e) |
| AC-LEAVE-08 | unit + e2e | The block end as a start is a boundary (unit); the exact message is shown (e2e) |
| AC-LEAVE-09 | unit + e2e | End before start is a boundary (unit); the exact message is shown (e2e) |
| AC-LEAVE-10 | e2e | The exact message is shown and no request appears in "My requests" |
| AC-LEAVE-11 | e2e | The kept dates and hours are visible in the form; "My requests" has no new record |
| AC-LEAVE-12 | e2e | The exact message is shown on screen |
| AC-LEAVE-13 | database + e2e | No overlapping record may be stored even outside the pages (database); the exact message is shown (e2e) |
| AC-LEAVE-14 | database + e2e | Another employee's records must not be readable at all (database); the list shows exactly 2 own records (e2e) |
| AC-LEAVE-15 | unit + e2e | Not counting the gap is a calculation (unit); the record with 1 h 30 min is visible (e2e) |
| AC-LEAVE-16 | database + e2e | "All records or none" is a rule of storing (database); the message and no new records are visible (e2e) |
| AC-LEAVE-17 | e2e | The comment is visible on both records |
| AC-LEAVE-18 | unit + e2e | 500 vs 501 characters is a boundary (unit); the exact message is shown (e2e) |
| AC-LEAVE-19 | unit + e2e | Hours across two blocks are a boundary (unit); the exact message listing both blocks is shown (e2e) |
| AC-LEAVE-20 | unit + e2e | The end of the first block as a start is a boundary (unit); the exact message is shown (e2e) |
| AC-LEAVE-21 | e2e | The message for Saturday and the one record for Friday are visible |
| AC-LEAVE-22 | unit + e2e | The start of the next block as the end is a boundary and the gap is not counted (unit); the record with 3 h is visible (e2e) |
| AC-DECIDE-01 | database + e2e | A manager must not read records of other positions at all (database); the list on screen does not show them (e2e) |
| AC-DECIDE-02 | e2e | Both statuses are visible on screen |
| AC-DECIDE-03 | e2e | The status "Rejected" is visible to the employee in "My requests" |
| AC-DECIDE-04 | e2e | The exact message is shown and the status on screen is unchanged |
| AC-DECIDE-05 | e2e | The status and the comment are visible to the employee |
| AC-DECIDE-06 | e2e | The new hours, the status and the record in the manager's list are visible |
| AC-DECIDE-07 | database + e2e | Only the first decision may be stored, even when both arrive at once (database); the exact message is shown (e2e) |
| AC-DECIDE-08 | database | A direct attempt bypassing the pages; only "nothing changed" can be proven, and that is a data rule |
| AC-DECIDE-09 | database | A direct attempt bypassing the pages; only "nothing changed" can be proven, and that is a data rule |
| AC-DECIDE-10 | e2e | The status and the history line naming the administrator are visible |
| AC-DECIDE-11 | e2e | The counts in "Awaiting decision" and the record in "History" are visible |
| AC-DECIDE-12 | database + e2e | An approved record must not change even outside the pages (database); the exact message is shown (e2e) |
| AC-DECIDE-13 | e2e | The status "Cancelled" and the manager's list without the record are visible |
| AC-DECIDE-14 | e2e | Empty state: the exact message is shown on screen |
| AC-DECIDE-15 | e2e | The history line with time, person and decision is visible |
| AC-DECIDE-16 | e2e | The new hours are visible to the employee and the manager |
| AC-DECIDE-17 | database + e2e | "Cancelled" is final even outside the pages (database); the status and no way to resubmit are visible (e2e) |
| AC-DECIDE-18 | database + e2e | A manager must not decide their own record even outside the pages (database); the exact message is shown (e2e) |
| AC-DECIDE-19 | e2e | The status "Approved" is visible |
| AC-DECIDE-20 | e2e | The comment is visible to the manager |
| AC-DECIDE-21 | database + e2e | Only the hours may change, even outside the pages (database); the exact message is shown (e2e) |
| AC-DECIDE-22 | database + e2e | No overlapping record may be stored even outside the pages (database); the exact message is shown (e2e) |
| AC-DECIDE-23 | e2e | The status and the comment are visible to the employee |
| AC-DECIDE-24 | e2e | The status and the comment are visible to the employee |
| AC-BALANCE-01 | e2e | All the numbers are visible on the balances page |
| AC-BALANCE-02 | unit + e2e | 1.00 day is a calculation (unit); the numbers are visible (e2e) |
| AC-BALANCE-03 | unit + e2e | 150 of 420 min rounded to 0.36 is a calculation with rounding (unit); the numbers are visible (e2e) |
| AC-BALANCE-04 | unit + e2e | Only approved records count — a calculation rule (unit); the unchanged numbers are visible (e2e) |
| AC-BALANCE-05 | unit + e2e | One date never counts more than 1.00 — a boundary (unit); the numbers are visible (e2e) |
| AC-BALANCE-06 | unit + e2e | 75 of 495 min is a calculation (unit); the numbers are visible (e2e) |
| AC-BALANCE-07 | e2e | The unchanged numbers are visible after rejecting |
| AC-BALANCE-08 | unit + e2e | The unpaid part 0.50 is a calculation (unit); the exact warning is shown (e2e) |
| AC-BALANCE-09 | unit + e2e | The suggested split is a calculation (unit); the exact warning is shown to the manager (e2e) |
| AC-BALANCE-10 | database + e2e | Both balances must change together when the record is approved (database); the numbers are visible (e2e) |
| AC-BALANCE-11 | unit | The date 2027-09-01 cannot be reached on production today; the school-year carry-over is a calculation with a fixed date |
| AC-BALANCE-12 | database + e2e | Balances of an employee outside the manager's positions must not be readable at all (database); nothing is shown on screen (e2e) |
| AC-BALANCE-13 | database + e2e | Another employee's balances must not be readable at all (database); nothing is shown on screen (e2e) |
| AC-BALANCE-14 | unit + e2e | The balance at decision time is a calculation (unit); the exact warning is shown (e2e) |
| AC-BALANCE-15 | unit + e2e | 0.36 − 0.20 = 0.16 is a calculation with rounding (unit); the exact warning is shown (e2e) |
| AC-BALANCE-16 | database + e2e | A record over the balance must not be approved without the split, even outside the pages (database); the exact message is shown (e2e) |
| AC-BALANCE-17 | unit + e2e | One warning for the total of the request is a calculation (unit); the exact single warning is shown (e2e) |
| AC-BALANCE-18 | e2e | The type, status and split are visible in "My requests" |
| AC-BALANCE-19 | unit + e2e | A split with 0.00 paid is a boundary (unit); the split and the numbers are visible (e2e) |
| AC-BALANCE-20 | database + e2e | A manager of any position reads the full balance (database); the same numbers are visible to both (e2e) |
| AC-BALANCE-21 | unit | Dates in 2027 cannot be reached on production today; a late approval across school years is a calculation with fixed dates |
| AC-BALANCE-22 | unit | Dates in 2027 cannot be reached on production today; splitting a request across school years is a calculation with fixed dates |
