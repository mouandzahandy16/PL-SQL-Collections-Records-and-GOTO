# PL-SQL-Collections-Records-and-GOTO
Repository: PL/SQL — Collections, Records, and GOTO

Course: Database development with PL/SQL (INSY 831)

Overview

This repository contains a complete assignment: a well-defined problem demonstrating PL/SQL collections (associative arrays, nested tables, VARRAYs), records (table-based, user-defined, cursor-based), and the use of GOTO statements. It includes:
	•	a clear problem statement
	•	SQL/PLSQL source files (scripts) that implement the solution
	•	test scripts and sample outputs
	•	documentation and assessment checklist for the instructor

Use this repository during the next class assessment.

⸻

Problem definition (assigned project)

Title: Department Salary Manager — analytics and corrections

Scenario: The HR department stores employee salary history per month. You must implement PL/SQL code to:
	1.	Load salary data for employees from a sample employees and salaries dataset (supplied or simulated). Salaries are monthly amounts.
	2.	Use three different collection types to store and process salary data:
	•	A VARRAY to store the last N monthly salaries for an employee (N fixed = 6)
	•	A nested table to collect all current-year bonuses for a department (variable length)
	•	An associative array (index-by table) to map employee_id —> computed metrics (e.g., average salary)
	3.	Define and use record types to hold employee metadata and aggregated results:
	•	A user-defined record employee_summary_rec containing employee_id, name, dept_id, avg_salary, last_salary
	•	A table-based record using %ROWTYPE when reading directly from employees table
	4.	Demonstrate cursor-based fetching into records for per-department processing.
	5.	Include a controlled use of GOTO to handle a retry mechanism when a non-fatal validation fails for a salary value (example: negative salary encountered — attempt a correction step; if correction fails, jump to cleanup label). This must be documented and justified.
	6.	Produce textual output using DBMS_OUTPUT.PUT_LINE with a summary per employee and per-department metrics.

Constraints / rules:
	•	The VARRAY must have an upper bound of 6 and be initialized correctly.
	•	The nested table must support deleting elements and checking .EXISTS when iterating.
	•	Use exception blocks and ensure cursors are closed on errors.
	•	Use GOTO only where it improves clarity for an explicit control-flow pattern (e.g., central cleanup or retry jump). Avoid spaghetti code.

Deliverables: PL/SQL scripts, README (this file), a test harness script, and a short assessment checklist.

⸻

Repository structure (suggested)

plsql-dept-salary-manager/
├─ README.md                      <- this document
├─ problem_description.md         <- concise project description (for students)
├─ src/
│  ├─ setup_sample_data.sql       <- create tables and seed sample rows
│  ├─ solution_main.sql           <- main PL/SQL block implementing requirements
│  ├─ pkg_salary_manager.sql      <- (optional) package with procedures/functions
│  └─ utils.sql                   <- helper procedures (print_collection, safe_fetch, etc.)
├─ tests/
│  ├─ run_all_tests.sql           <- runs scripts and shows expected output
│  └─ test_cases.md               <- list of test cases and expected results
└─ docs/
   └─ assessment_checklist.md     <- rubric & checklist used by instructor


⸻

Key files included (what to open first)
	•	src/setup_sample_data.sql — creates employees and salaries tables (if not available) and inserts sample rows.
	•	src/solution_main.sql — the main PL/SQL anonymous block showing collections, records, and GOTO usage.
	•	tests/run_all_tests.sql — run this to exercise the scripts and view DBMS_OUTPUT.
	•	docs/assessment_checklist.md — quick checklist for grading.

⸻

Example PL/SQL implementation (files in src/) — notes only

Note: The full scripts are supplied within this repository. Below is a summary of the key parts present in src/solution_main.sql.
	1.	Types and records declared
	•	TYPE salary_varray_t IS VARRAY(6) OF NUMBER;
	•	TYPE bonus_table_t IS TABLE OF NUMBER;
	•	TYPE emp_metric_map IS TABLE OF NUMBER INDEX BY PLS_INTEGER;
	•	TYPE employee_summary_rec IS RECORD (emp_id NUMBER, emp_name VARCHAR2(100), dept_id NUMBER, avg_salary NUMBER, last_salary NUMBER);
	2.	Sample flow
	•	Cursor c_depts loops over departments.
	•	Inside, cursor c_emps loops over employees in the department; for each employee:
	•	Load the last 6 months of salaries into a salary_varray_t variable.
	•	Compute average and last salary; store into an employee_summary_rec instance.
	•	Append any bonuses encountered into bonus_table_t.
	•	Store computed avg salary in emp_metric_map(emp_id).
	•	If a salary value is invalid (e.g., NULL or < 0), call a validate_and_correct internal procedure; if it can’t correct, use GOTO cleanup to break out to central cleanup.
	3.	Use of GOTO
	•	A single label retry_salary demonstrates a controlled retry attempt for a corrupt salary value.
	•	A cleanup: label ensures cursors are closed and temporary collections are freed.
	4.	Exception handling
	•	Clear WHEN NO_DATA_FOUND and WHEN OTHERS branches with helpful DBMS_OUTPUT.PUT_LINE of SQLERRM.

⸻

How to run (step-by-step)
	1.	Open SQL*Plus / SQLcl / SQL Developer connected to the target schema.
	2.	Run @src/setup_sample_data.sql to create demo tables and seed data (skip if you already have employees & salaries).
	3.	Run SET SERVEROUTPUT ON SIZE 1000000 (or equivalent) to see DBMS_OUTPUT.
	4.	Run @src/solution_main.sql.
	5.	Optional: Run @tests/run_all_tests.sql for additional assertions and expected output.

⸻

Sample output (what you should see)
	•	Per-department summary lines such as:

Department 10: processed 3 employees. Department average salary: 5233.33
Employee 101 - Alice: avg_salary=5200, last_salary=5300
Employee 102 - Bob: avg_salary=5100, last_salary=5000

	•	Warnings when an invalid salary was corrected or when a GOTO jump sent execution to cleanup.

⸻

Assessment checklist (docs/assessment_checklist.md)
	•	Uses VARRAY with upper bound of 6 correctly
	•	Uses nested table and demonstrates .DELETE and .EXISTS
	•	Uses associative array (INDEX BY) and demonstrates indexing by employee_id or integer
	•	Defines and uses user-defined record and table-based (%ROWTYPE) record
	•	Cursor-based record fetching implemented
	•	GOTO used in a justified, clearly documented, and limited way (no spaghetti code)
	•	Proper exception and cursor cleanup
	•	Scripts runnable with SET SERVEROUTPUT ON and produce readable output
	•	Documentation complete and organized
