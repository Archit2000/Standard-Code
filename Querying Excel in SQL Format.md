Certainly! Below is the **fully integrated official documentation** including your previously provided code plus the new enhancements (simplified sources input, base path support, and alias/sheet auto-inference). This can be used for internal distribution, repos, or wiki pages.

---

# 📘 Excel SQL Query Utility – Documentation

---

## Overview

This Python module enables executing SQL queries directly on Excel sheets using `pandasql`. It supports multiple sheets/files, join queries, and includes built-in safety checks, column sanitization, and warnings for risky SQL patterns.

Ideal for business intelligence, analytics, and automation tasks where Excel data needs SQL-style querying without importing to a database.

---

## Full Source Code

```python
import os
import pandas as pd
import re
from pandasql import sqldf

def safe_load_excel(file, sheet=None):
    """
    Loads an Excel sheet into a DataFrame with sanitized and unique column names.
    If `sheet` is None, loads the first sheet by default.
    Raises FileNotFoundError or ValueError if file or sheet is missing.
    """
    if not os.path.exists(file):
        raise FileNotFoundError(f"File not found: {file}")

    xls = pd.ExcelFile(file)

    if sheet is None:
        sheet = xls.sheet_names[0]

    if sheet not in xls.sheet_names:
        raise ValueError(f"Sheet '{sheet}' not found in {file}. Available: {xls.sheet_names}")

    df = pd.read_excel(file, sheet_name=sheet)
    df.columns = deduplicate_columns([re.sub(r"[() ]", "", col) for col in df.columns])
    return df

def deduplicate_columns(cols):
    """
    Returns a list of unique column names by appending suffixes (_1, _2, ...) to duplicates.
    """
    seen = {}
    new_cols = []
    for col in cols:
        if col not in seen:
            seen[col] = 1
            new_cols.append(col)
        else:
            count = seen[col]
            new_col = f"{col}_{count}"
            while new_col in seen:
                count += 1
                new_col = f"{col}_{count}"
            seen[col] = count + 1
            seen[new_col] = 1
            new_cols.append(new_col)
    return new_cols

def analyze_query_warnings(query):
    """
    Analyzes the SQL query for common risky patterns.
    Returns a list of warnings.
    """
    warnings = []

    cleaned_query = re.sub(r"\s+", " ", query.lower()).strip()

    if "select *" in cleaned_query:
        warnings.append("⚠️ Avoid using SELECT * — explicitly name columns to reduce risk.")

    if cleaned_query.startswith("select") and "where" not in cleaned_query and "limit" not in cleaned_query:
        warnings.append("⚠️ No WHERE or LIMIT clause found — may result in full-table scan.")

    if " join " in cleaned_query and " on " not in cleaned_query:
        warnings.append("⚠️ JOIN used without ON clause — this will produce a Cartesian product.")

    if cleaned_query.startswith("delete from") and "where" not in cleaned_query:
        warnings.append("❌ DELETE without WHERE — dangerous, this would delete all rows.")

    if cleaned_query.startswith("update") and "where" not in cleaned_query:
        warnings.append("❌ UPDATE without WHERE — dangerous, this would affect all rows.")

    return warnings

def normalize_sources(sources, base_path=None):
    """
    Converts user-friendly sources input into a list of dicts with keys:
    'file' (full path), 'sheet' (optional), and 'alias'.

    Supported input formats per source:
    - dict with keys 'file', optional 'sheet', optional 'alias'
    - tuple/list with (file, alias)
    - tuple/list with (file, sheet, alias)

    `base_path` defaults to current working directory.
    """
    base_path = base_path or os.getcwd()
    normalized = []

    for s in sources:
        if isinstance(s, dict):
            file = s.get("file")
            sheet = s.get("sheet", None)
            alias = s.get("alias", None)
        elif isinstance(s, (tuple, list)):
            if len(s) == 2:
                file, alias = s
                sheet = None
            elif len(s) == 3:
                file, sheet, alias = s
            else:
                raise ValueError("Source tuple/list must have 2 or 3 elements")
        else:
            raise TypeError("Each source must be dict or tuple/list")

        if not alias:
            alias = os.path.splitext(os.path.basename(file))[0]

        full_path = os.path.join(base_path, file)
        normalized.append({
            "file": full_path,
            "sheet": sheet,
            "alias": alias
        })

    return normalized

def query_excel_sources(sources, query, test_only=False, base_path=None):
    """
    Execute SQL query on one or more Excel sheets.

    Parameters:
    - sources: List of Excel sources in flexible formats (dict or tuple).
    - query: SQL query string using aliases as table names.
    - test_only: If True, runs query on first 5 rows for dry-run validation.
    - base_path: Base directory for Excel files (default is current working directory).

    Returns:
    - DataFrame with query results.

    Prints warnings and sample outputs in test mode.
    Raises exceptions with helpful debugging info on failure.
    """
    local_vars = {}
    normalized_sources = normalize_sources(sources, base_path=base_path)

    try:
        warnings = analyze_query_warnings(query)
        if warnings:
            print("🚨 SQL Warnings:")
            for w in warnings:
                print(f"  - {w}")
            print()

        for source in normalized_sources:
            df = safe_load_excel(source['file'], source['sheet'])
            local_vars[source['alias']] = df.head(5) if test_only else df

        if test_only:
            print("🧪 Dry-Run Mode: Testing SQL query on sample data...")

        result = sqldf(query, local_vars)

        if test_only:
            print("✅ SQL syntax looks valid.")
            print("Sample Output:")
            print(result.head())

        return result

    except Exception as e:
        print("❌ SQL Query Test Failed")
        print(f"Error: {e}")
        print("Available Tables and Columns:")
        for alias, df in local_vars.items():
            print(f"\n== {alias} ==")
            print(df.columns.tolist())
        raise
```

---

## Function Descriptions

### `safe_load_excel(file, sheet=None)`

* Loads Excel sheet into a pandas DataFrame.
* Sanitizes column names (removes spaces and parentheses).
* Deduplicates columns by appending suffixes.
* Defaults to first sheet if none specified.
* Raises exceptions if file or sheet is missing.

### `deduplicate_columns(cols)`

* Ensures unique column names by suffixing duplicates.
* Returns a list of unique column names.

### `analyze_query_warnings(query)`

* Basic SQL query pattern analysis.
* Flags use of `SELECT *`, missing `WHERE`/`LIMIT`, joins without `ON`, dangerous `UPDATE`/`DELETE`.
* Returns list of warnings.

### `normalize_sources(sources, base_path=None)`

* Converts user input (tuple or dict) into normalized dicts with absolute file paths.
* Infers alias from filename if not provided.
* Defaults to current working directory for file paths.

### `query_excel_sources(sources, query, test_only=False, base_path=None)`

* Loads all Excel sheets into DataFrames.
* Performs query warnings check.
* Runs SQL query on loaded DataFrames using `pandasql`.
* Supports `test_only` dry run mode for syntax and logic validation on sample data.
* Provides detailed error information and available table columns on failure.

---

## Usage Example

```python
sources = [
    ("employees.xlsx", "emp"),  # file + alias, first sheet default
    {"file": "departments.xlsx", "sheet": "Sheet1", "alias": "dept"}  # full dict form
]

query = """
SELECT emp.Name, dept.DepartmentName
FROM emp
JOIN dept ON emp.DeptID = dept.ID
WHERE emp.Salary > 60000
"""

# Dry-run validation on sample data
result_test = query_excel_sources(sources, query, test_only=True)

# Full query execution on entire sheets
result_full = query_excel_sources(sources, query, test_only=False)

print(result_full)
```

---

## Notes and Best Practices

* Always use `test_only=True` for initial validation of your SQL query.
* Explicitly specify columns in `SELECT` to avoid unintentional heavy queries.
* Avoid `DELETE` or `UPDATE` statements as they will not modify Excel files and are flagged as dangerous.
* Excel files should be in the same folder as the script or specify `base_path` accordingly.
* Use aliases consistently in SQL queries to reference loaded Excel sheets.

---

## Recommended Project Structure

```
/excel_sql/
   query_utils.py       # This module
/tests/
   test_query_utils.py  # Unit tests
/examples/
   example_query.py     # Example scripts and usage
```

---

If you need, I can also help generate this documentation in markdown or prepare Sphinx-style docstrings for auto-documentation.

Let me know how you want to proceed!
