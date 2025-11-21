# 📋 PHP CLI Assignment – Attendance Reconciliation Tool
(Part 1 Required • Part 2 Bonus)

## 🧾 Background

Your company tracks employee attendance using tap-in / tap-out logs and manages leave records separately.

HR needs a **daily attendance reconciliation report** to identify:

- Who came on time
- Who came but with attendance issues
- Who did not come and did not apply leave

You must build a **PHP CLI script** (`cli.php`) that reads two CSV files and prints a daily attendance status for all employees.

---

## 📂 Files Provided

You will be given **two CSV files**.

### 1. `attendance.csv`

| employeeId | date       | tapIn | tapOut |
|------------|------------|-------|--------|
| E001       | 2024-09-02 | 09:05 | 18:00  |
| E002       | 2024-09-02 | 09:30 | 18:15  |
| E004       | 2024-09-05 | 13:55 | 18:05  |

Notes:
- Time format: `HH:MM`
- If an employee has **no row** for a date → they **did not come**

---

### 2. `leave.csv`

| employeeId | leaveDate  | leaveType | timeStart | timeEnd |
|------------|------------|-----------|-----------|---------|
| E002       | 2024-09-03 | full-day  |           |         |
| E004       | 2024-09-04 | half-am   |           |         |
| E005       | 2024-09-05 | time-off  | 11:00     | 13:30   |

Leave types:
- `full-day`: 9:00–18:00
- `half-am`: 9:00–13:00
- `half-pm`: 14:00–18:00
- `time-off`: custom timeStart–timeEnd

Assumption: All input is valid (e.g., time-off does not overlap lunch 13:00–14:00).

---

## 🎯 Part 1 (Required) – Daily Attendance Report

Your script must print **one section per date**, in this format:

```
📆 Date: 2024-09-01 (Sunday)
────────────────────────────────────────────
[OK]     E001    Tap In: -        Tap Out: -        no leave needed
[ERROR]  E002    Tap In: 09:30    Tap Out: 18:15
[OK]     E004    Tap In: -        Tap Out: -        full-day
[MIA]    E005    Tap In: -        Tap Out: -        no leave applied


📆 Date: 2024-09-02 (Monday)
────────────────────────────────────────────
[MIA]    E001    Tap In: -        Tap Out: -        no leave applied
[MIA]    E002    Tap In: -        Tap Out: -        no leave applied
[OK]     E004    Tap In: 13:55    Tap Out: 18:05
[OK]     E005    Tap In: -        Tap Out: -        time-off (11:00–13:30)

📆 Date: 2024-09-03 (Tuesday)
────────────────────────────────────────────
[MIA]    E001    Tap In: -        Tap Out: -        no leave applied
[MIA]    E002    Tap In: -        Tap Out: -        no leave applied
[OK]     E004    Tap In: 13:55    Tap Out: 18:05    half-am
[MIA]    E005    Tap In: -        Tap Out: -        no leave applied

..... rest of the days for whole of 2024-09 month

```

Employee names are not required.

---

## 🧾 Required 3-State Logic

For each employee on each date, determine exactly one of:

### ✔ OK
Employee’s attendance or leave is acceptable.

Examples:
- Tap-in ≤ 09:00 and tap-out ≥ 18:00
- Attendance matches partial leave (half-am, half-pm, time-off)
- Full-day leave (attendance optional)

**Note for candidates:**  
If an employee has valid leave but still comes to work, treat it as **OK**.  
The company is happy if the employee works on leave.  
No validation needed.

---

### ❌ ERROR
Employee **came**, but attendance does NOT match any valid rule.

Examples:
- Late tap-in with no half-am / time-off covering the morning
- Early tap-out with no half-pm / time-off covering the afternoon
- Wrong attendance vs leave (e.g., tap-in 13:55 but leaveType = half-pm)

---

### ❌ MIA (Missing In Action)
Employee:
- Has **no attendance**
- Has **no leave**

---

## 🧠 Required Function

Your logic must be inside a function:

```php
function isEmployeePerfectForDay(
    string $employeeId,
    string $date,
    ?array $attendanceForDay,
    ?array $leaveForDay
): string {
    // return "OK", "ERROR", or "MIA"
}
```

---

## 💡 Helpful Hints

### Hint 1 — Parsing CSV

Use `fgetcsv()`:

```php
/**
 * Reads a CSV file and returns an array of associative rows.
 *
 * Example return:
 * [
 *   ["employeeId" => "E001", "date" => "2024-09-02", "tapIn" => "09:05", "tapOut" => "18:00"],
 *   ["employeeId" => "E002", "date" => "2024-09-02", "tapIn" => "09:30", "tapOut" => "18:15"],
 * ]
 */
function readCsv(string $filename): array
{
    if (!file_exists($filename)) {
        throw new RuntimeException("CSV file not found: {$filename}");
    }

    $rows = [];
    $handle = fopen($filename, 'r');

    if (!$handle) {
        throw new RuntimeException("Unable to open CSV file: {$filename}");
    }

    // Read header row
    $headers = fgetcsv($handle);

    if (!$headers) {
        fclose($handle);
        throw new RuntimeException("CSV file is empty or invalid: {$filename}");
    }

    // Clean BOM from first header if present
    $headers[0] = preg_replace('/^\xEF\xBB\xBF/', '', $headers[0]);

    while (($data = fgetcsv($handle)) !== false) {
        // Skip blank lines
        if (count($data) === 1 && trim($data[0]) === '') {
            continue;
        }

        $row = [];

        foreach ($headers as $i => $header) {
            // Assign null if column is missing
            $row[$header] = $data[$i] ?? null;
        }

        $rows[] = $row;
    }

    fclose($handle);
    return $rows;
}
```

### Hint 2 — Create lookup tables

```
$attendanceByDate[$date][$employeeId] = [...];
$leaveByDate[$date][$employeeId] = [...];
```

### Hint 3 — Collect ALL Employees & Dates

Extract from BOTH CSV files:

- employeeId
- date / leaveDate

Then:
- Make unique lists
- Sort them
- Loop dates → loop employees → call `isEmployeePerfectForDay()`

---

## ⭐ Part 2 (Bonus)

Optional enhancements:

### `--summary`
Totals per employee:
```
E001: OK=5  ERROR=1  MIA=0
```

### `--employee=E002`
Show only that employee.

### `--output=json`
Output summary as JSON.

---

## 📦 Deliverables

Submit:
- `cli.php`
- Any helper files
- Must include your implementation of `isEmployeePerfectForDay()`

---

## 📝 Notes

- Must run on **PHP 8.x**
- No frameworks required
- Code readability matters
