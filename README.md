<div align="center">

# BTMS - Banking Transaction Management System
## Database Testing Project (Manual Testing - INSERT / UPDATE)

![Test cases](https://img.shields.io/badge/test%20cases-103-blue) ![Passed](https://img.shields.io/badge/passed-68-brightgreen) ![Failed](https://img.shields.io/badge/failed-34-red) ![Blocked](https://img.shields.io/badge/blocked-1-yellow) ![Defects](https://img.shields.io/badge/defects-17-orange) ![DB](https://img.shields.io/badge/DB-MySQL%208.0-4479A1)

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Abdallah%20Meaad-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/abdallah-meaad) [![Portfolio](https://img.shields.io/badge/Portfolio-Visit%20Portfolio-black?style=for-the-badge&logo=googlechrome&logoColor=white)](https://abdallahmeaad.github.io/My_CV)

**Abdallah Ahmed Meaad** - QA / QC Engineer (Manual & Automation Testing)  
Phone: 01022656255

</div>

---

## 1. Overview
Manual database test of the BTMS Business Requirements Document: 7 tables (Customers, Accounts, Transaction_Categories, Transactions, Fees, Branches, Employees), validating **data integrity, business rules, relational consistency, constraints and triggers** using **INSERT and UPDATE statements only**.

The tests were designed from the BRD and executed against the **real database** (`btms-db.sql`, exported from MySQL 8.0.43 - 7 lower-case tables, 6 foreign keys, 3 CHECK constraints, 0 triggers), loaded **unchanged**. Every test is a manual SQL script run in a SQL client, inside `START TRANSACTION ... ROLLBACK`, so each test starts from the same baseline data.

## 1.1 Test data identity
All positive/negative test statements that insert a customer use the tester's own details as test data: **full_name = `Abdallah Meaad`**, **phone = `01022656255`** (e-mail `test.user@example.com`). Every statement runs inside `START TRANSACTION ... ROLLBACK`, so nothing is stored permanently.

## 2. Deliverables
| File | Content |
|---|---|
| `BTMS_Database_Testing_Abdallah_Meaad.xlsx` | The whole project in one workbook (see section 3) |
| `README.md` | This file |

## 3. Workbook sheets
| Sheet | What it contains |
|---|---|
| **Summary** | Live dashboard (formulas): totals, pass rate, weighted score, results by module, defects by severity, charts, GO / NO-GO, assumptions |
| **Test Cases** | 103 manual test cases: ID, module, type, priority, weight, preconditions, steps, SQL, expected result, actual result, Status dropdown, linked Bug ID |
| **Bug Reports** | 17 defects: severity/priority, description, reproduction SQL, expected vs actual, impact, suggested fix, status dropdown |
| **DB Schema** | Tables, columns, types, keys and constraints of the real database, mapped to the BRD rules |
| **DB ERD** | ERD of the real database (MySQL Workbench) + the 6 foreign-key relationships |
| **DB Seed Data** | Baseline data used by every test |
| **DB Scripts** | Copy-paste scripts taken from `btms-db.sql`: create the database, then load the seed data (verified identical to the dump) |

## 4. How to run the tests manually
1. Open **DB Scripts**, copy *Script 1 (Schema)* and run it in MySQL Workbench / DBeaver / MySQL CLI, then copy and run *Script 2 (Seed data)*.
2. Open **Test Cases**. For each row follow *Manual Test Steps* and run the SQL from *SQL to Execute*.
3. Compare the result with *Expected Result*, write what happened in *Actual Result* and choose *Pass / Fail / Blocked* in **Status** (the Summary sheet updates automatically).
4. For a failure, add or update the defect in **Bug Reports** and write its ID in the **Bug ID** column.

Requirements: MySQL 8.0.16+ (tested on 8.0.46). Older MySQL versions parse but **ignore** CHECK constraints, so the CHECK-related tests would fail for the wrong reason. Table names are lower-case, so the SQL also works on case-sensitive (Linux) servers.

## 5. Results (executed on MySQL 8.0.46, STRICT mode, real btms_db)
| Total | Pass | Fail | Blocked | Pass rate | Weighted score |
|---|---|---|---|---|---|
| 103 | 68 | 34 | 1 | 66.0% | 102 / 159 = 64.2% |

Release recommendation: **NO-GO**.

Key findings:
- The delivered DDL contains **no triggers**, although the BRD relies on them: balances never change after a transaction and fees are never calculated.
- Overdrafts are accepted (withdrawal or transfer above the balance).
- `CHECK(x > 0)` on nullable columns is bypassed by `NULL` (balance, transaction amount, fee amount).
- Not enforced: numeric-only phone, fee <= 10% of the transaction, receiver required for transfers, daily transfer limit.

## 6. Defect list
| ID | Severity | Title |
|---|---|---|
| BUG-001 | Critical | Balances are never updated by Deposit / Withdrawal / Transfer / Bill Payment (balance triggers missing) |
| BUG-002 | Critical | Withdrawal / Transfer / Bill Payment above the available balance is accepted as 'Success' |
| BUG-003 | Critical | Fees are not calculated automatically for transfers or failed transactions |
| BUG-004 | High | Accounts.balance accepts NULL (insert, omitted value and update) |
| BUG-005 | High | Transactions.amount accepts NULL (CHECK(amount > 0) is bypassed) |
| BUG-006 | High | Fees.amount accepts NULL (CHECK(amount > 0) is bypassed) |
| BUG-007 | High | Customers.phone accepts letters and symbols (numeric-only rule not enforced), on insert and update |
| BUG-008 | High | Fee larger than 10% of the transaction amount is accepted (insert and update) |
| BUG-009 | High | A Transfer can be inserted without a receiver account |
| BUG-010 | High | Daily transfer limit is not enforced (and its value is not defined in the BRD) |
| BUG-011 | Medium | Transfer from an account to itself is accepted |
| BUG-012 | Medium | Fee rules are not enforced: duplicate 'Failed Transaction Fee', failed-fee on a successful transaction, transfer-fee on a non-transfer |
| BUG-013 | Medium | Amount of a completed (Success) transaction can be edited |
| BUG-014 | Medium | Branches.location accepts NULL although it is a mandatory field |
| BUG-015 | Medium | Empty / blank strings satisfy 'mandatory' text columns (full_name, phone, category_name) |
| BUG-016 | Low | Weak data-quality validation: future date of birth, malformed e-mail, sub-cent money precision |
| BUG-017 | Medium | Schema gap: no link between Employees and transaction approvals, so the 'cannot delete approver' rule is impossible |

## 7. Test design techniques
Equivalence partitioning, boundary value analysis, negative testing, business-rule / state testing, error guessing and structural checks through `information_schema`.

## 8. Notes and limitations
- Executed on MySQL 8.0.46 (Linux, `lower_case_table_names=0`) after loading the real `btms-db.sql` dump (server 8.0.43). Both are the 8.0 series, so constraint behaviour and error codes (e.g. 3819 = CHECK violated) are the same.
- The delivered database already uses lower-case snake_case table names, so the earlier naming defect (BUG-018, PascalCase names from the BRD DDL) does not exist and was removed; TC-NFR-06 now passes.
- Suggested fixes for the CHECK/NOT NULL defects were verified on a scratch copy of the database: `MODIFY ... NOT NULL` keeps the existing CHECK constraints and the new CHECKs fire as expected.
- Where the BRD is silent (daily limit value, fee formula, meaning of "external transfer") the assumptions listed in the **Summary** sheet were used.
- Out of scope: DELETE, concurrency and atomicity of transfers. TC-EMP-06 is *Blocked* because the schema has no link between employees and transaction approvals (BUG-017).

---
<div align="center">Prepared by <b>Abdallah Ahmed Meaad</b> - QA / QC Engineer - Tel: 01022656255 - 2026-08-15</div>
