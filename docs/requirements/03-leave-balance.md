# Requirement: employees and managers see leave balances, and leave beyond the balance counts as Unpaid leave

<!--
Do not rename the ### section headings: developers compare versions by them.
The repo is public: no real client names, people's names or their data.
-->

### Why

Today the Sick leave and Personal leave balances are counted by hand in a spreadsheet, and absences of a few hours are
counted inconsistently. The employee does not know how many days are left, and the manager approves leave without
seeing that the balance is already used up. With this requirement the portal itself counts used and remaining days
from approved records, consistently for whole days and hours, and warns when a request exceeds the balance.

### Users and roles

- Employee (`employee`) — sees only their own balances; sees a warning when a request exceeds the balance.
- Responsible manager (`manager`) — sees the balances of employees of their own positions; sees a warning on the
  record and decides whether to approve it with a split into paid and unpaid parts.
- Administrator (`admin`) — sees the balances of all employees.

### Main scenario

1. For each leave type the employee sees: how many days are granted for the school year, how many were carried over
   from the previous one, how many are used and how many remain (for types without a limit — used only).
2. The employee submits a request; if it exceeds the balance, the employee sees a warning but can still submit.
3. The manager sees a warning on the record and a suggested split: a paid part up to the balance and an unpaid part
   for the rest.
4. The manager approves the record with the split or rejects it.
5. After approval the balance decreases; records with other statuses do not change the balance.

### Acceptance criteria

- AC-BALANCE-01: Given an employee with no approved records in school year 2026–2027 and no carried-over days. When the employee views their balances. Then the employee sees: "Sick leave" — granted 18.00, used 0.00, remaining 18.00; "Personal leave" — granted 18.00, used 0.00, remaining 18.00; "Unpaid leave" and "Family celebration" — used 0.00 only.
- AC-BALANCE-02: Given an employee with a Sick leave balance of 18.00 and a main position with the schedule Tue 08:30–15:30. When the manager approves a "Sick leave" record for the whole day 2026-11-03. Then used is 1.00, remaining 17.00.
- AC-BALANCE-03: Given an employee with a Sick leave balance of 18.00 and a single position with the schedule Tue 08:30–15:30 (7 h). When the manager approves a "Sick leave" record on 2026-11-03 from 09:00 to 11:30 (2 h 30 min). Then used is 0.36 (150 min of 420 min), remaining 17.64.
- AC-BALANCE-04: Given an employee with a Personal leave balance of 18.00 and a whole-day "Personal leave" record with status "Awaiting decision". When the employee views their balances. Then "Personal leave" — used 0.00, remaining 18.00.
- AC-BALANCE-05: Given an employee with a main position (Mon 08:30–15:30) and an additional position (Mon 15:45–17:00). When the managers approve both whole-day "Personal leave" records on 2026-11-02. Then used is 1.00, not 2.00.
- AC-BALANCE-06: Given an employee with a main position (Mon 08:30–15:30, 420 min) and an additional position (Mon 15:45–17:00, 75 min). When the manager approves a whole-day "Personal leave" record for the additional position only on 2026-11-02. Then used is 0.15 (75 min of the employee's 495 working minutes that day).
- AC-BALANCE-07: Given an employee with a Sick leave balance of 18.00 and a whole-day "Sick leave" record with status "Awaiting decision". When the manager rejects this record. Then "Sick leave" — used 0.00, remaining 18.00.
- AC-BALANCE-08: Given an employee with a Personal leave balance of 0.50 and a position with the schedule Tue 08:30–15:30. When the employee requests "Personal leave" for the whole day 2026-11-03. Then the request is created, and the employee sees the warning "Your Personal leave balance is 0.50 day. Another 0.50 day will be counted as Unpaid leave".
- AC-BALANCE-09: Given a whole-day "Personal leave" record with status "Awaiting decision" of an employee whose "Personal leave" shows used 17.50, remaining 0.50. When the manager opens the record. Then the manager sees the warning "This request exceeds the Personal leave balance (0.50 day). Suggested: 0.50 day Personal leave, 0.50 day Unpaid leave".
- AC-BALANCE-10: Given the record from AC-BALANCE-09 and the employee's "Unpaid leave" used 0.00. When the manager approves it with the suggested split. Then "Personal leave" — used 18.00, remaining 0.00; "Unpaid leave" — used 0.50.
- AC-BALANCE-11: Given an employee who has 5.00 days of Sick leave left on 2027-08-31 from school year 2026–2027. When the employee views their balances on 2027-09-01. Then "Sick leave" in school year 2027–2028 — granted 18.00, carried over 5.00, remaining 23.00.
- AC-BALANCE-12: Given employee teacher1@example.test has no positions managed by manager1@example.test. When manager1@example.test tries to view the balances of teacher1@example.test. Then the balances are not shown.
- AC-BALANCE-13: Given employee teacher1@example.test. When teacher1@example.test tries to view another employee's balances. Then the balances are not shown.
- AC-BALANCE-14: Given an employee whose "Personal leave" shows remaining 1.50, a position with the schedule Mon–Fri 08:30–15:30, and two whole-day "Personal leave" records on 2026-11-03 and 2026-11-04 with status "Awaiting decision". When the manager approves the 2026-11-03 record and then opens the 2026-11-04 record. Then the manager sees the warning "This request exceeds the Personal leave balance (0.50 day). Suggested: 0.50 day Personal leave, 0.50 day Unpaid leave".
- AC-BALANCE-15: Given an employee whose "Personal leave" shows remaining 0.20 and a single position with the schedule Tue 08:30–15:30 (420 min). When the employee requests "Personal leave" on 2026-11-03 from 09:00 to 11:30 (0.36 day). Then the request is created, and the employee sees the warning "Your Personal leave balance is 0.20 day. Another 0.16 day will be counted as Unpaid leave".
- AC-BALANCE-16: Given the record from AC-BALANCE-09. When the manager tries to approve it entirely as "Personal leave" without the split. Then the record is not approved, and the manager sees the message "The Personal leave balance is 0.50 day: approve with the suggested split or reject".

### Out of scope

- The administrator changing the number of days per year (18) in settings — slice WS10; in this slice it is 18 days.
- Returning days to the balance after an approved record is cancelled — slice WS11 (together with cancellation).
- Reports with leave hours and substitute minutes — slice WS9.
- Limits on Unpaid leave and Family celebration: they are only counted, not limited.
- Any amounts of money: the portal counts only days and hours.

### Assumptions

- All dates and times are in Kyiv time (Europe/Kyiv).
- The school year runs from 1 September to 31 August of the next calendar year (confirmed by PM).
- "Sick leave" and "Personal leave" — 18.00 days per school year each, separately (confirmed by PM); the limit belongs to the employee,
  not to a position: all of an employee's positions draw from one balance.
- Unused days carry over to the next school year in full, with no cap.
- Whole day = 1.00 day. Part of a day = the record's minutes ÷ all of the employee's working minutes on that date
  across the schedules of all their positions (confirmed by PM). One date never counts as more than 1.00 day.
- The share of a day is calculated for each date separately and rounded half up to 2 decimal places (0.355 → 0.36);
  the balance is the sum of the rounded shares.
- Only records with status "Approved" reduce the balance.
- The warning to the employee uses the balance at the moment of submission; the warning and suggested split for
  the manager use the balance at the moment the manager opens and decides the record, after any approvals made
  in between.
- Carried-over days at the start of the first school year for testing are entered by the developer as test data.

### Open questions

None.

### Change log

- 2026-10-19 — created
- 2026-10-19 — added AC-BALANCE-14 and an assumption: the manager's warning uses the balance at decision time — PM's own decision
- 2026-10-19 — added AC-BALANCE-15: exceeding the balance with a partial day — PM's own decision
- 2026-10-19 — AC-BALANCE-10: "0.50 higher than before" replaced with a concrete starting value and result — PM's own decision
- 2026-10-19 — added AC-BALANCE-16; open question "approve entirely as paid" answered: no, only the split or reject — PM's own decision
- 2026-10-19 — open question answered: 18 days of Sick leave and 18 days of Personal leave per school year — confirmed in Assumptions — PM's own decision
- 2026-10-19 — open question answered: the share of a day is calculated from the combined schedule of all the employee's positions on that date (as in AC-BALANCE-06) — confirmed in Assumptions — PM's own decision
- 2026-10-19 — open question answered: school year 1 September–31 August — confirmed in Assumptions — PM's own decision
- 2026-10-19 — all ACs: keywords temporarily switched to the template's Ukrainian form, then back to Given/When/Then; meaning and IDs unchanged — PM's own decision
- 2026-10-19 — Checked against the checklist: items 1–20 yes. Fixed: added AC-BALANCE-14, -15, -16 (balance at decision time, partial day, split only); gave AC-BALANCE-10 concrete numbers. Kept on purpose: 16 ACs, more than the 6–10 guideline; AC-BALANCE-01 checks one balance view with several numbers. AI chat: no.
- 2026-10-19 — document fully in English, section headings and change-log entries included — PM's own decision
