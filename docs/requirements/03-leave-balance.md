# Вимога: employees and managers see leave balances, and leave beyond the balance counts as Unpaid leave

<!--
Do not rename the ### section headings: they match the "Вимога" form and /requirements-sync relies on them.
The repo is public: no real client names, people's names or their data.
-->

### Навіщо

Today the Sick leave and Personal leave balances are counted by hand in a spreadsheet, and absences of a few hours are
counted inconsistently. The employee does not know how many days are left, and the manager approves leave without
seeing that the balance is already used up. With this requirement the portal itself counts used and remaining days
from approved records, consistently for whole days and hours, and warns when a request exceeds the balance.

### Користувачі й ролі

- Employee (`employee`) — sees only their own balances; sees a warning when a request exceeds the balance.
- Responsible manager (`manager`) — sees the balances of employees of their own positions; sees a warning on the
  record and decides whether to approve it with a split into paid and unpaid parts.
- Administrator (`admin`) — sees the balances of all employees.

### Основний сценарій

1. For each leave type the employee sees: how many days are granted for the school year, how many were carried over
   from the previous one, how many are used and how many remain (for types without a limit — used only).
2. The employee submits a request; if it exceeds the balance, the employee sees a warning but can still submit.
3. The manager sees a warning on the record and a suggested split: a paid part up to the balance and an unpaid part
   for the rest.
4. The manager approves the record with the split or rejects it.
5. After approval the balance decreases; records with other statuses do not change the balance.

### Критерії приймання

- AC-BALANCE-01: Дано an employee with no approved records in school year 2026–2027 and no carried-over days. Коли the employee views their balances. Тоді the employee sees: "Sick leave" — granted 18.00, used 0.00, remaining 18.00; "Personal leave" — granted 18.00, used 0.00, remaining 18.00; "Unpaid leave" and "Family celebration" — used 0.00 only.
- AC-BALANCE-02: Дано an employee with a Sick leave balance of 18.00 and a main position with the schedule Tue 08:30–15:30. Коли the manager approves a "Sick leave" record for the whole day 2026-11-03. Тоді used is 1.00, remaining 17.00.
- AC-BALANCE-03: Дано an employee with a Sick leave balance of 18.00 and a single position with the schedule Tue 08:30–15:30 (7 h). Коли the manager approves a "Sick leave" record on 2026-11-03 from 09:00 to 11:30 (2 h 30 min). Тоді used is 0.36 (150 min of 420 min), remaining 17.64.
- AC-BALANCE-04: Дано an employee with a Personal leave balance of 18.00 and a whole-day "Personal leave" record with status "Awaiting decision". Коли the employee views their balances. Тоді "Personal leave" — used 0.00, remaining 18.00.
- AC-BALANCE-05: Дано an employee with a main position (Mon 08:30–15:30) and an additional position (Mon 15:45–17:00). Коли the managers approve both whole-day "Personal leave" records on 2026-11-02. Тоді used is 1.00, not 2.00.
- AC-BALANCE-06: Дано an employee with a main position (Mon 08:30–15:30, 420 min) and an additional position (Mon 15:45–17:00, 75 min). Коли the manager approves a whole-day "Personal leave" record for the additional position only on 2026-11-02. Тоді used is 0.15 (75 min of the employee's 495 working minutes that day).
- AC-BALANCE-07: Дано an employee with a Sick leave balance of 18.00 and a whole-day "Sick leave" record with status "Awaiting decision". Коли the manager rejects this record. Тоді "Sick leave" — used 0.00, remaining 18.00.
- AC-BALANCE-08: Дано an employee with a Personal leave balance of 0.50 and a position with the schedule Tue 08:30–15:30. Коли the employee requests "Personal leave" for the whole day 2026-11-03. Тоді the request is created, and the employee sees the warning "Your Personal leave balance is 0.50 day. Another 0.50 day will be counted as Unpaid leave".
- AC-BALANCE-09: Дано a whole-day "Personal leave" record with status "Awaiting decision" of an employee whose "Personal leave" shows used 17.50, remaining 0.50. Коли the manager opens the record. Тоді the manager sees the warning "This request exceeds the Personal leave balance (0.50 day). Suggested: 0.50 day Personal leave, 0.50 day Unpaid leave".
- AC-BALANCE-10: Дано the record from AC-BALANCE-09 and the employee's "Unpaid leave" used 0.00. Коли the manager approves it with the suggested split. Тоді "Personal leave" — used 18.00, remaining 0.00; "Unpaid leave" — used 0.50.
- AC-BALANCE-11: Дано an employee who has 5.00 days of Sick leave left on 2027-08-31 from school year 2026–2027. Коли the employee views their balances on 2027-09-01. Тоді "Sick leave" in school year 2027–2028 — granted 18.00, carried over 5.00, remaining 23.00.
- AC-BALANCE-12: Дано employee teacher1@example.test has no positions managed by manager1@example.test. Коли manager1@example.test tries to view the balances of teacher1@example.test. Тоді the balances are not shown.
- AC-BALANCE-13: Дано employee teacher1@example.test. Коли teacher1@example.test tries to view another employee's balances. Тоді the balances are not shown.
- AC-BALANCE-14: Дано an employee whose "Personal leave" shows remaining 1.50, a position with the schedule Mon–Fri 08:30–15:30, and two whole-day "Personal leave" records on 2026-11-03 and 2026-11-04 with status "Awaiting decision". Коли the manager approves the 2026-11-03 record and then opens the 2026-11-04 record. Тоді the manager sees the warning "This request exceeds the Personal leave balance (0.50 day). Suggested: 0.50 day Personal leave, 0.50 day Unpaid leave".
- AC-BALANCE-15: Дано an employee whose "Personal leave" shows remaining 0.20 and a single position with the schedule Tue 08:30–15:30 (420 min). Коли the employee requests "Personal leave" on 2026-11-03 from 09:00 to 11:30 (0.36 day). Тоді the request is created, and the employee sees the warning "Your Personal leave balance is 0.20 day. Another 0.16 day will be counted as Unpaid leave".
- AC-BALANCE-16: Дано the record from AC-BALANCE-09. Коли the manager tries to approve it entirely as "Personal leave" without the split. Тоді the record is not approved, and the manager sees the message "The Personal leave balance is 0.50 day: approve with the suggested split or reject".

### Поза межами

- The administrator changing the number of days per year (18) in settings — slice WS10; in this slice it is 18 days.
- Returning days to the balance after an approved record is cancelled — slice WS11 (together with cancellation).
- Reports with leave hours and substitute minutes — slice WS9.
- Limits on Unpaid leave and Family celebration: they are only counted, not limited.
- Any amounts of money: the portal counts only days and hours.

### Припущення

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

### Відкриті питання

Немає

### Журнал змін

- 2026-10-19 — created
- 2026-10-19 — added AC-BALANCE-14 and an assumption: the manager's warning uses the balance at decision time — власне рішення PM
- 2026-10-19 — added AC-BALANCE-15: exceeding the balance with a partial day — власне рішення PM
- 2026-10-19 — AC-BALANCE-10: "0.50 higher than before" replaced with a concrete starting value and result — власне рішення PM
- 2026-10-19 — added AC-BALANCE-16; open question "approve entirely as paid" answered: no, only the split or reject — власне рішення PM
- 2026-10-19 — open question answered: 18 days of Sick leave and 18 days of Personal leave per school year — confirmed in Assumptions — власне рішення PM
- 2026-10-19 — open question answered: the share of a day is calculated from the combined schedule of all the employee's positions on that date (as in AC-BALANCE-06) — confirmed in Assumptions — власне рішення PM
- 2026-10-19 — open question answered: school year 1 September–31 August — confirmed in Assumptions — власне рішення PM
- 2026-10-19 — усі AC: ключові слова Given/When/Then замінено на Дано/Коли/Тоді за форматом шаблону, зміст і ID без змін — власне рішення PM
- 2026-10-19 — Перевірено за чеклістом: 1–20 так. Виправлено: додано AC-BALANCE-14, -15, -16 (залишок у момент рішення, частина дня, лише з поділом); AC-BALANCE-10 отримав конкретні числа. Лишили свідомо: 16 AC — більше за орієнтир 6–10; AC-BALANCE-01 перевіряє один екран залишків із кількома числами. AI-чат: ні.
