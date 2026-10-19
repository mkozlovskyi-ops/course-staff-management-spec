Spec 3 of 3 — leave balance. Depends on: spec-1-leave-request, spec-2-leave-decision. Used by: none.

# Requirements Document

> Source: `docs/requirements/03-leave-balance.md` and `docs/product.md` (PM repo, 2026-10-19), split out of the
> combined spec `.claude/specs/leave-management-ws6/` (its Requirements 9–11). Every acceptance criterion keeps the PM's
> AC ID so tests and conformance tables can be named after it. If this document and the PM text disagree, the PM text
> wins.

## Introduction

This spec covers the leave-balance part of slice WS6 of the school staff leave portal. Today Sick leave and Personal
leave balances are counted by hand in a spreadsheet, and absences of a few hours are counted inconsistently. With this
feature the portal counts granted, carried-over, used and remaining days per leave type and school year from approved
leave records, consistently for whole days and hours, warns the employee when a request exceeds the balance, and lets
the manager approve such a request only with a split into a paid part (up to the balance) and an Unpaid leave part (the
rest).

Leave requests and records come from spec-1-leave-request; decisions on records come from spec-2-leave-decision. This
spec adds balance counting to them and does not change their other rules.

## Alignment with Product Vision

- Success indicator in `docs/product.md`: "Sick leave and Personal leave balances are calculated automatically to
  0.01 day and match the approved records" → Requirements 1 and 2.
- Permission matrix row "See other employees' balances": employee — no; manager — only employees of own positions;
  admin — yes → Requirement 3.
- Payroll needs a complete and correct split between paid and unpaid absence; the portal counts only days and hours,
  never money (out of scope in `docs/product.md`).

## Business Rules (from the PM's assumptions)

These rules apply to every requirement below; they are confirmed by the PM and are not acceptance criteria themselves.

1. All dates and times are in Kyiv time (Europe/Kyiv).
2. The school year runs from 1 September to 31 August of the next calendar year; "2026–2027" starts on 2026-09-01.
3. Sick leave and Personal leave are each granted 18.00 days per school year, separately. The balance belongs to the
   employee, not to a position: all of an employee's positions draw from one balance.
4. Unpaid leave and Family celebration have no limit: they are only counted.
5. Unused days carry over to the next school year in full, with no cap. Carried-over days at the start of the first
   school year are entered by the developer as test data.
6. A whole day is 1.00 day. Part of a day is the record's minutes divided by all of the employee's working minutes on
   that date across the schedules of all their positions. One date never counts as more than 1.00 day.
7. The share of a day is calculated for each date separately and rounded half up to 2 decimal places (0.355 → 0.36);
   the balance is the sum of the rounded shares.
8. Only records with status "Approved" reduce the balance; records with any other status do not change it.
9. The employee's warning uses the balance at the moment of submission; the manager's warning and suggested split use
   the balance at the moment the manager opens and decides the record, after any approvals made in between.
10. The manager approves an over-balance record only with the suggested split, or rejects it; approving it entirely
    as paid leave is not possible.

## Requirements

### Requirement 1 — Balance calculation and view

**User Story:** As an employee, I want to see the granted, carried-over, used and remaining days for each leave type in
the school year, so that I know how much leave I have left without asking anyone.

#### Acceptance Criteria

1. **AC-BALANCE-01** WHEN an employee with no approved records in school year 2026–2027 and no carried-over days views
   their balances THEN the portal SHALL show "Sick leave" — granted 18.00, used 0.00, remaining 18.00; "Personal
   leave" — granted 18.00, used 0.00, remaining 18.00; and "Unpaid leave" and "Family celebration" — used 0.00 only.
2. **AC-BALANCE-02** WHEN the manager approves a "Sick leave" record for the whole day 2026-11-03 of an employee with a
   Sick leave balance of 18.00 and a main position with the schedule Tue 08:30–15:30 THEN the portal SHALL show
   "Sick leave" used 1.00, remaining 17.00.
3. **AC-BALANCE-03** WHEN the manager approves a "Sick leave" record on 2026-11-03 from 09:00 to 11:30 (2 h 30 min) of
   an employee with a Sick leave balance of 18.00 and a single position with the schedule Tue 08:30–15:30 (7 h) THEN
   the portal SHALL show "Sick leave" used 0.36 (150 min of 420 min), remaining 17.64.
4. **AC-BALANCE-04** IF an employee with a Personal leave balance of 18.00 has a whole-day "Personal leave" record with
   status "Awaiting decision" THEN the portal SHALL show "Personal leave" used 0.00, remaining 18.00 when the employee
   views their balances.
5. **AC-BALANCE-05** WHEN the managers approve both whole-day "Personal leave" records on 2026-11-02 of an employee
   with a main position (Mon 08:30–15:30) and an additional position (Mon 15:45–17:00) THEN the portal SHALL show
   "Personal leave" used 1.00, not 2.00.
6. **AC-BALANCE-06** WHEN the manager approves a whole-day "Personal leave" record on 2026-11-02 for the additional
   position only of an employee with a main position (Mon 08:30–15:30, 420 min) and an additional position
   (Mon 15:45–17:00, 75 min) THEN the portal SHALL show "Personal leave" used 0.15 (75 min of the employee's 495
   working minutes that day).
7. **AC-BALANCE-07** WHEN the manager rejects a whole-day "Sick leave" record with status "Awaiting decision" of an
   employee with a Sick leave balance of 18.00 THEN the portal SHALL show "Sick leave" used 0.00, remaining 18.00.
8. **AC-BALANCE-11** WHEN an employee who had 5.00 days of Sick leave left on 2027-08-31 from school year 2026–2027
   views their balances on 2027-09-01 THEN the portal SHALL show "Sick leave" in school year 2027–2028 — granted
   18.00, carried over 5.00, remaining 23.00.

### Requirement 2 — Over-balance warning and split into paid and Unpaid leave

**User Story:** As an employee and as a responsible manager, I want to be warned when a request exceeds the balance
and to have the excess counted as Unpaid leave, so that paid leave is never overspent and payroll gets a correct split.

#### Acceptance Criteria

1. **AC-BALANCE-08** WHEN an employee with a Personal leave balance of 0.50 and a position with the schedule
   Tue 08:30–15:30 requests "Personal leave" for the whole day 2026-11-03 THEN the portal SHALL create the request AND
   SHALL show the employee the warning "Your Personal leave balance is 0.50 day. Another 0.50 day will be counted as
   Unpaid leave".
2. **AC-BALANCE-15** WHEN an employee whose "Personal leave" shows remaining 0.20 and who has a single position with
   the schedule Tue 08:30–15:30 (420 min) requests "Personal leave" on 2026-11-03 from 09:00 to 11:30 (0.36 day) THEN
   the portal SHALL create the request AND SHALL show the employee the warning "Your Personal leave balance is 0.20
   day. Another 0.16 day will be counted as Unpaid leave".
3. **AC-BALANCE-09** WHEN the manager opens a whole-day "Personal leave" record with status "Awaiting decision" of an
   employee whose "Personal leave" shows used 17.50, remaining 0.50 THEN the portal SHALL show the manager the warning
   "This request exceeds the Personal leave balance (0.50 day). Suggested: 0.50 day Personal leave, 0.50 day Unpaid
   leave".
4. **AC-BALANCE-14** WHEN, for an employee whose "Personal leave" shows remaining 1.50, with a position scheduled
   Mon–Fri 08:30–15:30 and two whole-day "Personal leave" records on 2026-11-03 and 2026-11-04 with status "Awaiting
   decision", the manager approves the 2026-11-03 record and then opens the 2026-11-04 record THEN the portal SHALL show
   the manager the warning "This request exceeds the Personal leave balance (0.50 day). Suggested: 0.50 day Personal
   leave, 0.50 day Unpaid leave".
5. **AC-BALANCE-10** WHEN the manager approves the record from criterion 2.3 with the suggested split AND the
   employee's "Unpaid leave" used was 0.00 THEN the portal SHALL show "Personal leave" used 18.00, remaining 0.00, AND
   "Unpaid leave" used 0.50.
6. **AC-BALANCE-16** IF the manager tries to approve the record from criterion 2.3 entirely as "Personal leave"
   without the split THEN the portal SHALL NOT approve the record AND SHALL show the manager the message "The Personal
   leave balance is 0.50 day: approve with the suggested split or reject".

### Requirement 3 — Balance privacy

**User Story:** As an employee, I want my balances to be visible only to me, the managers of my positions and the
administrator, so that my absence history stays private.

#### Acceptance Criteria

1. **AC-BALANCE-12** IF employee teacher1@example.test has no positions managed by manager1@example.test AND
   manager1@example.test tries to view the balances of teacher1@example.test THEN the portal SHALL NOT show the
   balances.
2. **AC-BALANCE-13** IF teacher1@example.test tries to view another employee's balances THEN the portal SHALL NOT show
   the balances.

## Out of Scope

- The administrator changing the number of days per year (18) in settings — slice WS10.
- Returning days to the balance after an approved record is cancelled — slice WS11.
- Reports with leave hours and substitute minutes — slice WS9.
- Limits on Unpaid leave and Family celebration: they are only counted.
- Any amounts of money: the portal counts only days and hours.

## Non-Functional Requirements

### Code Architecture and Modularity

- **Single Responsibility Principle:** day-share, school-year and split arithmetic live in pure functions with no I/O,
  separate from data access and from the screens that display balances.
- **Modular Design:** balance counting is one module reused by the balances screen, the employee's request form and
  the manager's record card; none of them computes balances on its own.
- **Dependency Management:** the balance feature depends on spec-1 (records, leave types) and spec-2 (decisions); it
  changes their behaviour only where a criterion above requires it.
- **Clear Interfaces:** the over-balance check is enforced on the server at decision time, so the rule holds for direct
  API calls as well as the UI.

### Precision and Rounding

- Day values are counted to 0.01 day: each date's share is rounded half up to 2 decimal places (0.355 → 0.36) and
  capped at 1.00 per date; balances are sums of rounded shares and never accumulate floating-point error.
- The same input always gives the same day values in the form preview, in warnings and in the stored balance.
- Day values are always displayed with exactly 2 decimals (0.50, 18.00).

### Performance

- One employee's balances for one school year are returned in a single request to the server and render within 1 s
  for an employee with up to 500 approved records.
- Opening a record card computes the warning without loading the employee's full record list into the browser.

### Security and Privacy

- Balances follow the permission matrix in `docs/product.md`: own balances for every role, employees of own positions
  for a manager, all employees for the administrator; this holds for direct API calls, not only in the UI
  (AC-BALANCE-12, -13).
- A viewer who may not see an employee's balances gets the same response as for a non-existent employee, so the portal
  does not reveal whether the employee exists.
- No salary, pay rate or money amount is stored or shown.
- The paid and unpaid days of every approved record stay visible to the same people who may see the record.

### Reliability and Consistency

- Concurrent approvals for the same employee (several managers, or one manager in two tabs) are serialised: the
  combined approvals never take paid days beyond the balance (AC-BALANCE-14).
- The over-balance check and the stored split are computed from the balance at the moment of the decision, never from
  values sent by the browser (AC-BALANCE-16).
- Rejecting, returning or cancelling a record that was never approved does not change any balance (AC-BALANCE-04, -07).
- The paid and unpaid days of an approved record are fixed at approval, so later schedule changes do not silently
  change past balances.

### Usability

- Balances show the current school year by default and are labelled "2026–2027" style.
- Types without a limit show only "used"; types with a limit show granted, carried over, used and remaining.
- The employee's warning does not block submission; the manager's warning shows the exact suggested split before the
  decision.
- Warning and error texts are exactly those quoted in the criteria.
