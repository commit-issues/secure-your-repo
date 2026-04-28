# Database Security

*Your database is where everything that matters lives. Treat it like it.*

---

!!! info "TL;DR"

    - Most database breaches are not sophisticated attacks — they are default settings left unchanged, credentials reused, and access controls never configured.
    - Encrypt your database at rest. If someone walks out with the file, they should walk out with nothing usable.
    - Never use the root or admin database user in your application code. Create a least-privilege user and use that.
    - Parameterized queries only. No exceptions. SQL injection is still one of the most common attack vectors in existence.
    - Every audit log entry is permanent. You do not delete logs. You do not edit logs. They are your evidence.
    - A backup you have never tested is not a backup. It is a false sense of security.

    Already handling encryption and access control? Jump to [The Audit Log →](#the-audit-log--append-only)

---

In 2021, a misconfigured database exposed the personal records of over 700 million LinkedIn users. In 2019, a single misconfigured MongoDB instance leaked 275 million Indian citizens' personal data — names, phone numbers, email addresses, employment history. In 2017, Equifax exposed the financial records of 147 million people. The technical root causes vary, but the pattern is consistent: the data was there, accessible, and nobody had done the work to protect it.

Your database is the end of the line. It is where credentials land after authentication, where transactions are recorded, where user data accumulates over time. Everything you built — the input validation, the encrypted secrets, the secure network calls — exists to protect what ends up here. This section is about making sure the database itself is not the thing that undoes all of it.

---

## Choosing the Right Database for Your Use Case

Before you can secure a database, you need to be running the right one for what you are building. The wrong choice does not just create security problems — it creates operational ones that make security harder to implement correctly.

### SQLite

SQLite is a file-based database. There is no server, no network connection, no authentication layer. The entire database lives in a single file on disk. Your application reads and writes that file directly.

This makes SQLite excellent for local tools, desktop applications, development environments, and anything that runs on a single machine without needing concurrent access from multiple processes.

What SQLite is not appropriate for: anything with multiple simultaneous users, anything that needs to be accessed over a network, anything running in a cloud environment where the database needs to scale independently of the application.

### PostgreSQL and MySQL

PostgreSQL and MySQL are server-based databases. They run as a separate process, accept network connections, handle concurrent users, and have full authentication and access control systems built in. These are the right choice for web applications, APIs, anything deployed to a server, and anything with more than one user.

The tradeoff is operational complexity. You are now running a server, managing credentials, configuring network access, and thinking about backups differently than you do with a file.

!!! tip "When in doubt, PostgreSQL"

    PostgreSQL has stronger data integrity guarantees, better support for complex queries, and a more active security patch process than MySQL. If you are starting a new project and are not sure which to use, PostgreSQL is the safer default.

### Local vs cloud-hosted

Cloud-hosted databases — AWS RDS, Supabase, PlanetScale, Railway — handle a significant amount of operational security for you: automated backups, encryption at rest by default, managed patching, network isolation. The tradeoff is that you are trusting a third party with your data and need to configure their access controls correctly.

Self-hosted databases give you full control and full responsibility. Every configuration decision is yours.

Neither is inherently more secure. Both have been breached. The difference is where the mistakes happen.

---

## Encryption at Rest — SQLCipher

Encryption in transit — the HTTPS and TLS we covered in the [Network Security →](network.md) section — protects your data while it is moving. Encryption at rest protects it while it is sitting still.

If someone physically takes your server, copies your disk, steals your backup, or gains filesystem access to your machine, an unencrypted database file gives them everything. Every user record, every credential, every piece of sensitive data — readable with any SQLite browser or database client, no password required.

In 2012, Dropbox was breached. The attackers obtained a database of hashed user passwords. Because the passwords were hashed, they were not immediately usable — but they were extractable and crackable offline, at the attacker's leisure, with no time pressure. Encryption at rest would not have prevented the breach, but it is the layer that makes offline extraction either impossible or prohibitively expensive.

### What SQLCipher is

SQLCipher is an open-source extension to SQLite that adds full database encryption using AES-256. Every page of the database file is encrypted before it is written to disk and decrypted when read. Without the key, the file is unreadable — it looks like random noise. With the key, it behaves exactly like a normal SQLite database.

The CVE Security Intelligence Monitor, an open-source security tool I built at `commit-issues/cve-security-monitor`, uses SQLCipher for exactly this reason: it runs locally, manages sensitive security data, and if the database file were ever exfiltrated, SQLCipher ensures it is useless without the key.

```python
# Python — connecting to a SQLCipher database
# macOS / Linux
# pip3 install sqlcipher3

# Windows
# pip install sqlcipher3

import sqlcipher3

def open_database(db_path, key):
    conn = sqlcipher3.connect(db_path)
    conn.row_factory = sqlcipher3.Row
    conn.execute(f"PRAGMA key='{key}'")
    conn.execute("PRAGMA cipher_page_size=4096")
    conn.execute("PRAGMA kdf_iter=256000")
    conn.execute("PRAGMA cipher_hmac_algorithm=HMAC_SHA512")
    conn.execute("PRAGMA cipher_kdf_algorithm=PBKDF2_HMAC_SHA512")
    return conn
```

### Key management

The encryption is only as strong as how you protect the key. A key hardcoded in your source code, committed to git, or stored in the same place as the database defeats the entire purpose.

```python
# Wrong — key in source code
conn.execute("PRAGMA key='mysecretkey123'")

# Correct — key from environment variable
import os
db_key = os.environ.get("DB_ENCRYPTION_KEY")
if not db_key:
    raise RuntimeError("DB_ENCRYPTION_KEY not set")
conn.execute(f"PRAGMA key='{db_key}'")
```

Store the key in your environment, a secrets manager, or a separate secrets file that is never committed. See [Credential Management →](credentials.md) for the full approach.

!!! warning "Encrypting an existing unencrypted database"

    If you are adding SQLCipher to a database that already exists and contains data, you cannot just add the PRAGMA key and call it done. You need to use `sqlcipher_export()` to create a new encrypted copy. Doing this wrong can corrupt your data. Test the migration on a copy first. Never on production data directly.

---

## Authentication and Access Control

This is the section where most breaches actually happen. Not sophisticated exploits — just databases running with default credentials, or application code connecting as root because nobody set up a proper user.

### Never use the root or admin user in application code

The root user on a database has permission to do everything: read any table, drop any database, create or delete any user, modify any configuration. Your application almost certainly does not need any of that.

In 2017, the MongoDB ransomware attacks wiped tens of thousands of databases in a matter of days. The majority of affected databases were running with no authentication at all — the default MongoDB configuration at the time did not require a password. Attackers wrote scripts to scan for open MongoDB ports, connect without credentials, exfiltrate data, delete it, and leave a ransom note. No exploit required. Just a default configuration and an open port.

Create a dedicated user for your application with only the permissions it actually needs:

```sql
-- PostgreSQL — create a least-privilege application user

-- Create the user
CREATE USER app_user WITH PASSWORD 'use-a-strong-password-from-env';

-- Grant only what the application needs
GRANT CONNECT ON DATABASE your_database TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO app_user;

-- Do NOT grant DELETE unless your application explicitly needs it
-- Do NOT grant DROP, CREATE, or ALTER
-- Do NOT grant access to tables the application does not use
```

```sql
-- MySQL — create a least-privilege application user
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'use-a-strong-password-from-env';
GRANT SELECT, INSERT, UPDATE ON your_database.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
```

### Separate credentials for every environment

Your development database and your production database should never share credentials. If your dev environment is compromised — a malicious package, a compromised developer machine, a leaked dotfile — production credentials should not be reachable from it.

```
Development:  DB_USER=dev_user      DB_PASS=dev-only-password
Staging:      DB_USER=staging_user  DB_PASS=staging-only-password
Production:   DB_USER=prod_user     DB_PASS=strong-unique-production-password
```

Each environment connects to its own database with its own credentials. Never the same key across environments. See [Credential Management →](credentials.md) for how to manage this without losing your mind.

### Connection string security

A connection string packages your database host, port, username, password, and database name into a single value. It is convenient. It is also a single string that, if leaked, hands an attacker everything they need to connect.

```python
# Wrong — connection string in source code
conn = psycopg2.connect("postgresql://root:password123@localhost/mydb")

# Correct — assembled from environment variables
import os
conn = psycopg2.connect(
    host=os.environ["DB_HOST"],
    port=os.environ["DB_PORT"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASS"],
    dbname=os.environ["DB_NAME"]
)
```

Never put a connection string in source code. Never commit it to git. Never log it. Treat every component of it — especially the password — as a secret.

---

## Parameterized Queries — Non-Negotiable

SQL injection has been in the OWASP Top 10 most critical web application security risks for over fifteen years. It is not a new or exotic attack. It is one of the oldest and most well-documented vulnerabilities in existence, and it still takes down applications regularly because developers concatenate strings instead of using parameters.

This is covered in detail in [Input Handling & Validation →](input-validation.md). It belongs here too because it is a database security issue as much as it is an input issue.

In 2008, Heartland Payment Systems was breached via SQL injection. 130 million credit card numbers were stolen. It remains one of the largest data breaches in history. The technique used was known, documented, and preventable.

```python
# Wrong — string concatenation, injectable
user_input = request.args.get("username")
cursor.execute(f"SELECT * FROM users WHERE username = '{user_input}'")

# Correct — parameterized query
cursor.execute("SELECT * FROM users WHERE username = %s", (user_input,))
```

```python
# SQLite / SQLCipher — same principle
cursor.execute("SELECT * FROM cves WHERE severity_level = ?", (severity,))
```

The parameter is never interpreted as SQL. It is passed as data. The database never confuses it for a command. This is not optional — it is the baseline.

---

## The Audit Log — Append Only

An audit log is a permanent, chronological record of what happened in your database: who accessed what, what changed, when, and from where. It is not the same as application logging. Application logs help you debug. An audit log is evidence.

When the breach happens — and for a sufficiently complex application, the question is when, not if — your audit log is how you answer: what data was accessed, when did the attacker get in, what did they change, and who was affected. Without it, you are guessing. With it, you can reconstruct exactly what happened.

### The append-only principle

Audit logs must be append-only. You add entries. You never edit them. You never delete them. If an audit log can be modified, it is not evidence — it is a document that someone might have changed.

This matters because the first thing an attacker who gains database access will try to do is cover their tracks. If your audit log lives in the same database they just compromised, and they have write access to it, your evidence is gone.

```python
# Python — append-only audit log table setup
def setup_audit_log(conn):
    conn.execute("""
        CREATE TABLE IF NOT EXISTS audit_log (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
            event_type TEXT NOT NULL,
            user_id TEXT,
            ip_address TEXT,
            table_name TEXT,
            record_id TEXT,
            action TEXT NOT NULL,
            old_value TEXT,
            new_value TEXT,
            outcome TEXT NOT NULL
        )
    """)
    # Revoke delete and update permissions on this table
    # for the application user — inserts only
```

```python
# Logging an audit event
def log_audit_event(conn, event_type, user_id, action, table_name=None,
                    record_id=None, old_value=None, new_value=None,
                    outcome="success", ip_address=None):
    conn.execute("""
        INSERT INTO audit_log
        (event_type, user_id, ip_address, table_name, record_id,
         action, old_value, new_value, outcome)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
    """, (event_type, user_id, ip_address, table_name, record_id,
          action, old_value, new_value, outcome))
    conn.commit()
```

### What to log at the database level

```
Authentication events     — logins, failures, logouts, permission changes
Data access               — who read sensitive records and when
Data modification         — what changed, the old value, the new value, who changed it
Schema changes            — table creation, column changes, index modifications
Permission changes        — who was granted or revoked access to what
Failed queries            — especially repeated failures on the same record
```

You do not need to log every SELECT on every table. You do need to log access to anything sensitive — user records, payment data, credentials, PII — and every write operation without exception.

---

## Backups

In 2019, a misconfigured script at GitLab deleted 300GB of production database data in minutes. The engineer ran a database removal command on the wrong server. Six hours of data was gone. GitLab had five different backup systems. Four of them were not working correctly. One had data but it was six hours old.

GitLab recovered. They were transparent about what happened. But the incident illustrated exactly what backup failure looks like in practice: multiple systems, none tested, all assumed to be working.

### The rules

**Back up frequently.** How frequently depends on how much data loss is acceptable. For most applications, daily is the minimum. For anything handling financial transactions or user-generated content, more often.

**Encrypt your backups.** A backup is a copy of your database. Everything we said about encrypting the database applies equally to the backup. An unencrypted backup stored in S3 is an unencrypted database file accessible to anyone who gets into that bucket.

**Store backups separately.** A backup on the same server as the database it backs up is not a backup. If the server goes down, you lose both. Backups need to be in a separate location — a different machine, a different cloud region, offline storage.

**Test your restores.** This is the one most people skip. A backup file that has never been restored is a file that might restore correctly. You do not know until you try. Schedule restore tests. Actually run them. Confirm the data is intact and the application works against the restored database.

```bash
# PostgreSQL — automated backup with timestamp
pg_dump -U app_user -h localhost your_database | \
  gzip | \
  gpg --symmetric --cipher-algo AES256 \
  > backup_$(date +%Y%m%d_%H%M%S).sql.gz.gpg

# Store the encrypted backup offsite
# Never store the GPG passphrase next to the backup
```

```bash
# Test a restore — do this regularly, not just after something breaks
gpg --decrypt backup_20260101_120000.sql.gz.gpg | \
  gunzip | \
  psql -U app_user -h localhost your_database_restore_test
```

!!! warning "The backup you have never tested is not a backup"

    It is a file you are hoping will work when you need it most. Schedule a restore test. Put it in your calendar. Do it.

---

## Database in CI/CD

Your CI/CD pipeline runs tests. Those tests need a database. That database should never be your production database, and it should never use your production credentials.

In 2020, a developer at a major fintech company accidentally ran a test suite against a production database. The test suite included teardown scripts that wiped tables between tests. Several thousand live user records were deleted before anyone noticed. The company recovered the data from backups, but the incident required regulatory disclosure.

### Test databases

Use a separate database for tests. Either spin one up fresh for each CI run, or use a persistent test database that gets seeded with known data before each run and cleaned up after.

```python
# conftest.py — pytest setup for test database
import pytest
import os

@pytest.fixture(scope="session")
def test_db():
    # Use a separate test database — never production
    db_path = os.environ.get("TEST_DB_PATH", ":memory:")
    conn = get_connection(db_path)
    setup_schema(conn)
    seed_test_data(conn)
    yield conn
    conn.close()
    # Clean up if file-based
    if db_path != ":memory:":
        os.remove(db_path)
```

### Environment separation

```bash
# .env.test — never shares values with .env.production
DB_HOST=localhost
DB_NAME=myapp_test
DB_USER=test_user
DB_PASS=test-only-password
```

```bash
# .env.production — loaded only in production
DB_HOST=prod-db.internal
DB_NAME=myapp_production
DB_USER=prod_user
DB_PASS=strong-unique-production-password
```

Your CI pipeline should load `.env.test`. Your production deployment should load `.env.production`. They should never overlap. If your CI pipeline ever touches production credentials, something is wrong with your setup.

---

## Migrations — Doing Them Safely

A database migration is any change to the structure of your database — adding a table, adding a column, changing a data type, removing something that is no longer needed. Every significant application goes through dozens or hundreds of migrations over its lifetime.

Done wrong, a migration can corrupt data, take down production, or create a state that is impossible to roll back from. Done right, it is invisible to users.

### The rules for safe migrations

**Never modify production data directly.** Write a migration script. Run the script. The script is version controlled. The manual edit is not.

**Test migrations on a copy of production data first.** Not on staging with fake data — on an actual copy of what production looks like. Schema changes that work fine on small datasets can take hours or lock tables on large ones.

**Make migrations reversible.** Every migration should have a corresponding rollback. If the migration introduces a bug, you need to be able to undo it without data loss.

**Run migrations before deploying new code, not after.** New code that depends on a new column should be deployed after that column exists, not before. The order matters.

```python
# Python — simple migration runner
import os
import glob

def run_migrations(conn):
    # Create migrations table if it doesn't exist
    conn.execute("""
        CREATE TABLE IF NOT EXISTS schema_migrations (
            version TEXT PRIMARY KEY,
            applied_at TEXT DEFAULT (datetime('now', 'utc'))
        )
    """)

    # Find all migration files
    migration_files = sorted(glob.glob("migrations/*.sql"))

    for filepath in migration_files:
        version = os.path.basename(filepath).replace(".sql", "")

        # Skip already-applied migrations
        already_applied = conn.execute(
            "SELECT 1 FROM schema_migrations WHERE version = ?", (version,)
        ).fetchone()

        if already_applied:
            continue

        # Apply the migration
        with open(filepath, "r") as f:
            sql = f.read()

        conn.executescript(sql)
        conn.execute(
            "INSERT INTO schema_migrations (version) VALUES (?)", (version,)
        )
        conn.commit()
        print(f"Applied migration: {version}")
```

### Rolling back a bad migration

If a migration causes problems in production, you need a rollback plan before you run it — not after you discover the problem.

```sql
-- migrations/003_add_notification_type.sql
ALTER TABLE tool_alerts ADD COLUMN notification_type TEXT DEFAULT 'Desktop';

-- migrations/003_add_notification_type_rollback.sql
-- SQLite does not support DROP COLUMN directly before version 3.35
-- Create a new table without the column and copy data
CREATE TABLE tool_alerts_backup AS SELECT id, alert_type, created_at FROM tool_alerts;
DROP TABLE tool_alerts;
ALTER TABLE tool_alerts_backup RENAME TO tool_alerts;
```

Write the rollback script at the same time as the migration. Commit both. If you ever need the rollback, you will not have time to write it from scratch.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
