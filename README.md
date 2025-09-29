# 📋 PHP CLI Assignment – Attendance Reconciliation Tool

## 🧾 Background

Your company’s HR department tracks daily employee attendance using tap-in/out logs and manages leave records separately.

Each month, HR manually reconciles:

- Who came to work?
- Who was late?
- Who didn’t show up?
- Who was on approved leave?

You're tasked to automate this process by building a **PHP CLI script** to analyze two CSV files and generate a clear report.

---

## 📂 Files Provided

You will receive **two CSV files**:

### 1. `attendance.csv`

Daily tap-in and tap-out logs.

| employeeId | date       | tapIn | tapOut |
|------------|------------|-------|--------|
| E001       | 2024-09-02 | 09:05 | 18:00  |
| E002       | 2024-09-02 | 09:30 | 18:15  |

- Time format: 24-hour, `HH:MM`
- A missing record means the employee didn’t tap in/out.

### 2. `leave.csv`

Approved leave records.

| employeeId | leaveDate  | leaveType | timeStart | timeEnd |
|------------|------------|-----------|-----------|---------|
| E002       | 2024-09-03 | full-day  |           |         |
| E004       | 2024-09-04 | half-am   |           |         |
| E005       | 2024-09-05 | time-off  | 11:00     | 13:30   |

- `leaveType` can be:
    - `full-day` → entire work day (9am–6pm)
    - `half-am`  → morning leave (9am–1pm)
    - `half-pm`  → afternoon leave (2pm–6pm)
    - `time-off` → custom block (`timeStart` to `timeEnd`)

❗ Important Constraint:
The company designates lunch time from 1:00pm to 2:00pm. Therefore, time-off leave blocks cannot fall within 13:00–14:00.

---

## ✅ Part 1 – Daily Reconciliation Report

Write a PHP CLI script that:

1. **Reads and parses** both `attendance.csv` and `leave.csv`.
2. For each working day:
    - List all employees.
    - Show tap-in/out if present.
    - Indicate if the employee:
        - Was on approved leave (and what kind)
        - Was late (tap-in after 9:00am unless `half-pm` or `time-off`)
        - Did not show up (no attendance, no leave)
3. Print the results to console in a clean, grouped format per date.

### 🖨️ Sample Output Format

```
📆 Date: 2024-09-03 (Tuesday)
────────────────────────────────────────────
✔️  E001 (Alice)     Tap In: 09:05    Tap Out: 18:00
⚠️  E002 (Bob)       Tap In: 09:30    Tap Out: 18:15    Late
🔁  E004 (Diana)     Tap In: 13:55    Tap Out: 18:05    On Leave: half-am
❌  E005 (Ethan)     No Record        No Leave Applied
```

---

## ✅ Part 2 – Summary Reporting & CLI Options (Required)

### 1. `--summary` Flag (Per-Employee Summary)

```bash
php cli.php --summary
```

Output a monthly summary for each employee:

```
📊 Summary for September 2024
────────────────────────────────────────────
E001 - Alice
  ✔ Present:     5 days
  ⚠ Late:        2 days
  ❌ No-show:     0 days
  🔁 Leave:       1 full-day, 1 half-day
```

---

### 2. `--employee=E003` Filter

```bash
php cli.php --employee=E004
```

Show daily logs (and summary) only for that employee.

---

### 3. `--output=json` or `> file.txt`

Enable structured output:

```bash
php cli.php --summary --output=json
```

Outputs:

```json
[
  {
    "employeeId": "E001",
    "presentDays": 5,
    "lateDays": 2,
    "noShowDays": 0,
    "leaveDays": {
      "full-day": 1,
      "half-am": 0,
      "half-pm": 1
    }
  }
]
```

Or redirect output to a file:

```bash
php cli.php > report.txt
```

---

### 4. 📦 Company-Wide Totals (Required)

Always include overall stats at the bottom of the summary output:

```
📦 Company Totals
────────────────────────────────────────────
✔️  Total Present Days: 58
⚠️  Total Late Days:    11
❌  Total No-shows:     3
🔁  Total Leave Days:   10 (4 full, 4 half, 2 time-off)
```

---

## 📎 Deliverables

Submit:

- `cli.php` (entry script)
- Any helper classes or utilities

---

## 📝 Notes

- Code must run on **PHP 8.x**
- You may use plain PHP or small libraries for CLI argument parsing
- Output must be clear and grouped as described
