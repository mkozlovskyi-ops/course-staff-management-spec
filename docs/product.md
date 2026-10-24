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
