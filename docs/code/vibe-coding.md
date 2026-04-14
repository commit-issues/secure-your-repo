# Vibe Coding & AI-Assisted Development

> AI is not trained to secure your product. That is your job.

---

!!! abstract "TL;DR"
    - AI generates code that works. Working is not the same as secure.
    - This section is long because this topic deserves the full treatment.
    - If you want to skip to fixes, tools, and commands: [Jump to The Fix →](#the-fix-tools-and-commands)
    - Plan before you build. Security and privacy first. User experience above all.
    - Every AI-generated code block needs human review before it ships.
    - The tools in this section automate what your eyes miss.

---

AI-assisted development is not the problem. Shipping AI-generated code without understanding it is the reason most security incidents happen in projects built this way. Not in the GitHub settings. Not in the deployment pipeline. In the code itself — written fast, accepted without review, and pushed to production before it is secured and deemed safe — leaving it wide open to vulnerabilities

This section is long, and that is for good reason. The setup steps, the hardening, the branch protection — all of that protects the container. This section is about what goes inside the container and how to make it functional and secure.

---

## Before You Write a Single Line — Plan First

The single most underrated security practice is also the most basic one: knowing what you are building before you start building it. AI makes it tempting to skip this step entirely — you describe an idea, code appears, it runs, you ship. That pipeline produces working software and terrible security posture simultaneously.

Before you start building, consider pseudocode — and if you have never heard of it, pay attention because this matters.

Pseudocode is plain-language planning written before any real code exists. It describes what your program will do, step by step, in human language — not Python, not JavaScript, not any specific syntax. It is the architectural blueprint before the walls go up. Large companies use pseudocode to plan, confirm, align teams, implement correctly, and protect a project from scope creep and hidden assumptions. It forces you to think through the logic before you commit to an implementation.

Most people who vibe code have never encountered the concept because AI will just start writing code the moment you describe an idea. That speed feels productive. It is often the opposite — you are building before you know what you are building.

The author of this guide produces content and code under the name SudoCode — and the online presence and username across platforms is SudoChef. The GitHub handle is commit-issues, which says something too: not that commits are a problem, but that there are always too many of them because something is always being improved. In Unix and Linux systems, `sudo` grants superuser permissions — the ability to execute commands with root access, the highest level of system control. Even if you do not have those permissions by default, sudo is how you get them. That is the standard to hold yourself to: approach your code with that level of authority, ownership, and responsibility. Know what you are building. Own every decision in it. Have root-level understanding of your own project. That is what pseudocode enables.

Not planning is pure laziness, and it will reflect in the finished product. A project built without a plan is a project that grows unpredictably, breaks in unexpected places, and accumulates security debt in the dark.

**What to define before you write any code:**

```
1. What does this actually do?
   Write one sentence. If you cannot, you are not ready to build.
   "A tool that lets users upload photos and applies filters"
   is a sentence. "A platform" is not.
   If your one sentence needs ten qualifiers, break it into 
   smaller projects first.

2. Who uses it?
   Just you? A small team? The public? Children? Healthcare workers?
   The answer changes everything — authentication requirements,
   rate limiting, data handling obligations, accessibility needs,
   and the entire threat model shift based on who your user is.

3. What data does it touch?
   Does it store user data? Names, emails, locations?
   Payment details? Health records? Biometric data?
   Government IDs? Children's data?
   Each category has legal implications and security requirements
   that are not optional — regardless of whether you knew about
   them when you started building.

4. What can go wrong?
   If someone malicious used this, what would they try to do?
   If it crashed at peak usage, what would break?
   If it leaked data, whose data would it be and what would 
   happen to them as a result?
   If someone submitted garbage input, would it crash or continue?

5. What does success look like at scale?
   10 users? 10,000? 10 million?
   "It works" at 10 users means nothing at 10,000.
   Rate limiting, database indexing, connection pooling, caching,
   CDN — these are not features you add later.
   They are architecture decisions you make at the start.

6. What are the contingencies?
   What happens if the third-party API you depend on goes down?
   What happens if your database is unavailable for 30 minutes?
   What happens if a user submits malformed or malicious input?
   What happens if your cloud provider has an outage?
   These are not edge cases. They are production reality.
   Plan for them before they happen.

7. Who owns this and who is responsible for it?
   Solo project? Team? If a user's data is breached, who answers 
   for it? If the app goes down and people are depending on it,
   who fixes it and how fast?
   Responsibility does not disappear because you did not plan for it.

8. What is the monetization or sustainability model?
   Free forever? Freemium? Paid? API costs money.
   Server costs money. Storage costs money.
   A tool that works at zero users may bankrupt you at 10,000
   if you did not model the costs.

9. What are the legal and regulatory requirements?
   GDPR applies if you have users in the European Union.
   CCPA applies if you have users in California.
   HIPAA applies if you touch health data in the US.
   PCI-DSS applies if you handle payment card data.
   COPPA applies if your users could be children.
   If you do not know what these are — stop.
   Before you write a single line of code, understand which 
   regulations apply to your project. Study them. Have AI 
   break them down for you, then cross-reference with Google 
   or another browser to verify. Always ask AI to cite its 
   sources on legal and regulatory questions. Getting this 
   wrong is not a technical debt problem — it is a legal one.

10. What does the full user journey look like from start to finish?
    How does someone find the product? How do they sign up?
    What is the first thing they see? What does onboarding look like?
    How do they accomplish the core task? How do they get help?
    How do they leave or delete their account?
    Map every step before you build any of them.
```

**The security and privacy questions that must be answered before line one:**

```
→ Where does user data live and who can access it?
→ What is the minimum data we need to collect?
   (Collect less. Store less. Expose less.)
→ How do users authenticate?
→ What can authenticated users do that unauthenticated users cannot?
→ What can admins do that regular users cannot?
→ How do we handle a data breach if one happens?
→ What regulations apply? (GDPR, CCPA, HIPAA, PCI-DSS)
```

**User experience is not the last thing you think about — it is the first.**

UX and UI design is an entire professional field and area of study. People dedicate entire careers to it. Your competition takes it seriously. You should too — because user experience is what determines whether people actually use your product or abandon it after thirty seconds.

A secure application that nobody can use is not a secure application. Security that creates friction gets bypassed. Users who cannot figure out how to do something safely will find an unsafe way to do it. The best security is invisible — it protects without getting in the way.

Before you build, answer these questions:

```
→ What is the complete user journey from start to finish?
  How does a new user discover the product?
  How do they sign up? What does authentication look like?
  What is the first screen they see after logging in?
  How do they find the core feature? How do they use it?
  How do they manage their profile and settings?
  How do they get help when something goes wrong?
  How do they delete their account if they want to leave?
  Map every single step. Build none of them before you have the map.

→ Where will people primarily use this?
  Phone? Desktop? Laptop? Tablet?
  The answer changes your layout, your navigation, your 
  button sizes, your font sizes, and your entire interaction model.
  A mobile-first app and a desktop-first app are designed differently
  from the ground up — not the same thing with a responsive stylesheet.

→ What platforms and operating systems does it need to support?
  iOS? Android? Windows? macOS? Linux? All of them?
  Will it be a web app that runs in a browser?
  A native app that installs on a device?
  A desktop app that runs locally?
  Will it be in the App Store or Google Play?
  What does the download and installation experience look like?
  Or does it live locally on someone's machine and never go public?
  Each answer has different distribution, update, and security implications.

→ Is the interface obvious to someone who has never seen it before?
  The test: give it to someone who was not involved in building it.
  Watch them use it without helping them.
  Where they get confused is where you failed.

→ Are error messages helpful to users and safe from attackers?
  "Invalid credentials" is correct.
  "No account found with that email" tells an attacker which 
  emails are registered. Never confirm whether an account exists.

→ Does every security requirement have a UX solution that 
  does not punish the user?
  2FA should be smooth, not a wall.
  Session timeouts should warn before they happen.
  Password requirements should be shown upfront, not after failure.
```

Plan for security. Plan for privacy. Plan for users. Then build.

---

## What Vibe Coding Actually Is

"Vibe coding" is the practice of building software primarily by prompting an AI — describing what you want, accepting what it generates, prompting again when something breaks, and repeating until it works. The name comes from the feeling of it: you are vibing with the AI, iterating quickly, shipping fast.

It is genuinely powerful. It has lowered the barrier to building software dramatically. People who could never have written a web application five years ago are shipping working products today. That is not nothing.

It also produces a specific and well-documented category of problems — not because AI is bad at generating code, but because the workflow optimizes for speed and functionality while systematically skipping the steps that produce secure, maintainable software.

Understanding why this happens requires understanding what AI is actually doing when it writes code.

---

## What AI Cannot Know When It Writes Your Code

When you ask an AI to generate code, it draws on patterns from the vast amount of code it was trained on. It produces code that matches those patterns — code that looks like code that works, structured like code that works, and in many cases, code that does work.

What it cannot know — and this list will grow as technology advances, because these are structural limitations of how AI works, not bugs that will be patched:

```
→ Your specific environment
  What operating system, what Python version, what library
  versions are actually installed on your machine or server.
  macOS, Linux, and Windows handle paths, permissions, and 
  system calls differently. AI generates code for a generic
  environment that may not match yours.

→ Your users
  Who they are, what they will try to do, how they will
  misuse or abuse the system, their technical sophistication,
  their accessibility needs, their languages, their devices.

→ Your threat model
  What an attacker targeting your specific application would
  actually try. A financial app, a healthcare tool, a social
  platform, and a developer portfolio have completely different
  threat models. AI does not know which one you are building
  unless you explicitly tell it — and even then, it cannot
  reason about threats the way a security professional can.

→ What is currently safe
  AI has a knowledge cutoff. The library version it suggests
  as current may have a critical CVE published against it since
  training. The API pattern it recommends may have been 
  deprecated. The cryptographic approach it uses may have 
  been broken. It does not know. It cannot know.

→ Your data
  What is actually in your database, what your users data
  looks like in practice, what edge cases your real data 
  produces, how large your datasets will grow.

→ The regulatory environment you operate in
  GDPR, HIPAA, CCPA, PCI-DSS, COPPA, SOC2 — AI can describe
  these but cannot apply them to your specific situation with
  legal accuracy. Compliance is not a code problem.

→ The consequences of failure
  If this breaks, who is affected and how badly?
  If data leaks, whose data is it and what are the 
  downstream effects on real people?
  The AI has no stake in the answer.

→ Your infrastructure
  How your production environment is configured, what your
  network topology looks like, what your CI/CD pipeline does,
  what monitoring you have in place.

→ Your team and your processes
  Who reviews code, how often deployments happen, what your
  incident response looks like, whether anyone will actually
  rotate those API keys every 90 days.

→ The social and organizational context
  Who has access to what, who has left the company and still
  has credentials, what vendor relationships exist that
  create third-party risk.

→ Future vulnerabilities
  A library that is safe today may have a zero-day published
  tomorrow. AI-generated code that passes every scan today
  may be vulnerable next week. Security is not a state you
  achieve. It is a practice you maintain.
```

This is not a criticism of AI. It is a description of the tool. A hammer cannot know whether the nail is in the right place. The person holding the hammer is responsible for that. We are providing fixes and tools for the issues we can address today — but these will change as technology continues to advance. Treat this section as a living reference, not a final checklist.

---

## For Complete Beginners — "It Works" Is Not the Finish Line

If you have been building with AI and you have no computer science or security background, this section is written for you.

You asked AI to build something. It gave you code. You ran it. It worked. That feels like success — and in one sense it is. Getting something to run is genuinely hard and genuinely satisfying.

But "it works" answers one question: does this do what I described? It does not answer: is this safe? Is this stable under real usage? Could someone break this, steal data from it, or use it to harm your users?

"It works" also does not mean it will work when real people use it. Development testing is not production testing. Ten friends testing something on a Tuesday afternoon is not the same as ten thousand strangers using it simultaneously on a Monday morning. You do not know if it works live until it is live — and by then, bugs are user experiences, not development notes.

Large companies use beta programs for exactly this reason. They release to a limited audience first, collect feedback, find the bugs that only appear under real usage patterns, and fix them before the full release. There will be bugs. Assume it. Plan for it. Build feedback mechanisms into the product from the start.

Here is the analogy. If you have only ever used an instruction manual to assemble IKEA furniture, that does not mean you have the knowledge or capability to build a full functioning house. A house has electrical systems, plumbing, load-bearing structures, fire codes, building permits — each of which is a professional discipline that licensed experts study for years. You would not wire your own electrical panel after watching one YouTube video. The same principle applies here.

A working demo is IKEA furniture. A production application serving real users is the house. The gap between them is testing, hardening, monitoring, error handling, rate limiting, authentication, data protection, and every other discipline covered in this guide.

Before you go public: test it yourself end to end. Have people who were not involved in building it test it — friends, family, anyone willing to break things. Watch them use it. Fix what breaks. Then test again. Then go public.

"It works" means the furniture is assembled. Security and production-readiness mean it is safe to live in.

**The most dangerous sentence in security is: "I don't understand this code but it works."**

If you do not understand the code running in your application, you cannot defend it. You cannot explain what data it accesses, what it does with that data, or what happens when it receives unexpected input. You are flying blind — and an attacker who understands your code better than you do has a significant advantage.

**What to do when you don't understand code AI gave you:**

```
Step 1 — Ask the AI to explain it.
  "Explain this code to me line by line as if I have 
  never seen Python before."
  Read the explanation. If it still does not make sense,
  ask follow-up questions until it does.

Step 2 — Ask about the security implications.
  "What are the security risks in this code?"
  "What could go wrong if a malicious user sent 
  unexpected input to this function?"
  "Is there a more secure way to write this?"

Step 3 — Do not ship it until you understand it.
  This is the hard part. The temptation is to ship 
  and figure it out later. Later never comes.
  Understanding what you ship is not optional — it is 
  the minimum bar for responsible development.

Step 4 — If you still cannot understand it after 
  asking, it is the wrong tool for this job.
  Some things require more foundation than AI can give you
  in a single session. That is okay. Build the foundation
  before you build the feature.
```

---

## For Experienced Developers — The New Challenge

If you have been writing software for years, you already know how to write secure code. The challenge AI introduces is not a knowledge gap — it is a discipline gap.

AI moves faster than your review process. It generates in seconds what would take you hours to write. That speed is the point. It is also the problem.

When you write 20 lines yourself, you review every line because you wrote it. You know what it does. When AI generates 200 lines, the psychological pressure to accept it and move on is real — especially when it looks reasonable, the tests pass, and you have ten other things to do.

**The false confidence of passing tests:**

Tests passing on AI-generated code means the code does what the tests describe. Tests are written by the same AI that wrote the code, often testing the happy path only — the expected inputs, the expected outputs. They do not test what an attacker would send. They do not test the edge cases that only appear in production. They do not test the security properties that were never specified in the prompt.

Working tests on insecure code are not a safety net. They are a confidence trap.

**Technical debt velocity:**

In traditional development, technical debt accumulates at roughly the pace you write code. When you write a shortcut, you know you wrote a shortcut. With AI, debt accumulates at the pace AI generates code — which is ten to a hundred times faster. A week of vibe coding can produce months of technical debt that is invisible until it fails.

The debt that matters most from a security perspective is the debt you cannot see: the missing input validation, the unrotated credentials, the API endpoint with no authentication, the rate limiter that was never added because the AI never mentioned it.

**Subtle vulnerabilities that pass code review:**

The most dangerous AI-generated vulnerabilities are the ones that look right. Not the obvious hardcoded API key — those get caught. The subtle ones:

```python
# This looks fine. It is not.
query = f"SELECT * FROM users WHERE email = '{email}'"

# This is SQL injection. The email variable is user input.
# AI generates this pattern constantly because it works 
# in testing where email is always a valid email address.
# In production, email could be:
# ' OR '1'='1
# ...and now you have handed over your entire users table.

# The correct version:
query = "SELECT * FROM users WHERE email = ?"
cursor.execute(query, (email,))
```

The first version passes every functional test. The second version is the only one that is safe. AI knows both patterns — it produces whichever one matches the context of the prompt, and the prompt rarely specifies "make this SQL injection resistant."

---

## What AI Slop Looks Like — How to Spot It

"AI slop" is the informal term for AI-generated code that is functional but structurally poor — bloated, inconsistent, fragile, and difficult to maintain or secure. Industry leaders who are skeptical of vibe coding are not skeptical of AI — they are skeptical of unreviewed AI output being shipped to production.

Here is what it looks like:

**Unnecessary length:**

AI tends to be verbose. A function that should be 10 lines is 40. Logic that should be a single expression is five nested conditionals. This is not just aesthetic — longer code has more surface area, more places for bugs to hide, and more for a security reviewer to read and miss something in.

```python
# AI slop — 15 lines to do what 3 lines do
def is_valid_email(email):
    if email is None:
        return False
    if len(email) == 0:
        return False
    if '@' not in email:
        return False
    parts = email.split('@')
    if len(parts) != 2:
        return False
    if len(parts[0]) == 0:
        return False
    if len(parts[1]) == 0:
        return False
    if '.' not in parts[1]:
        return False
    return True

# Clean version — same logic, 3 lines
import re
def is_valid_email(email):
    return bool(email and re.match(r'^[^@]+@[^@]+\.[^@]+$', email))
```

**Old code left in when new code is added:**

Every time you prompt AI to fix or change something, it adds new code. The old code it replaced often stays — commented out, renamed, or just left there doing nothing. Dead code is not neutral. It is confusion, it is potential for a future developer to accidentally re-enable something broken, and it is attack surface.

**Spacing, indentation, and capitalization errors:**

In Python, indentation is syntax. A single misindented line changes what the code does — not just how it looks. AI-generated code frequently has inconsistent indentation, especially in long functions or when code was added across multiple prompts. These errors do not always cause obvious crashes. Sometimes they cause subtle logic errors that only appear under specific conditions.

```python
# This looks fine. The indentation error is subtle.
def process_payment(amount, user):
    if user.is_authenticated:
        if amount > 0:
            charge_card(amount)
            log_transaction(amount, user)
        return True  # This should be indented one more level
    return False

# The return True executes whether the charge succeeded or not,
# and whether amount > 0 or not. The indentation error means
# the payment logic is completely bypassed in some cases.
```

**Context decay — AI forgetting what it built:**

In a long session, AI loses track of earlier decisions. It rewrites a function you already wrote, using a different pattern. It adds a dependency you already removed. It creates a second version of something that already exists. The result is contradictory code — two functions doing the same thing differently, two authentication patterns that conflict, two error handlers that fight each other.

**Hallucinated libraries:**

AI sometimes references packages that do not exist, or exist but do not have the methods being called. The code looks correct, runs no syntax errors, and fails only at runtime — or worse, fails silently when the import happens to succeed but the method does not exist in the installed version.

---

## The Top Technical Failures — What Goes Wrong and Why

These are the failures that appear most consistently in AI-generated applications. Each one is a real, documented pattern with real consequences.

---

### Rate Limiting — Why Your App Dies at 10 Users

This is one of the most common ways vibe-coded applications fail in production. You build something, it works perfectly in development, you share it, ten people use it simultaneously, and it stops working.

Rate limiting is the practice of restricting how many requests a user or system can make in a given time period. Without it, a single user — or a bot — can make thousands of requests per second, overwhelming your server, exhausting your API quota, and potentially running up costs into the thousands of dollars in minutes.

AI almost never adds rate limiting unless you specifically ask for it. It is not part of the "happy path" that prompts describe. It is an operational concern that only becomes visible at scale.

```python
# No rate limiting — anyone can call this endpoint unlimited times
@app.route('/api/login', methods=['POST'])
def login():
    email = request.json['email']
    password = request.json['password']
    user = User.query.filter_by(email=email).first()
    if user and user.check_password(password):
        return jsonify({'token': generate_token(user)})
    return jsonify({'error': 'Invalid credentials'}), 401

# With rate limiting — 5 attempts per minute per IP
from flask_limiter import Limiter
limiter = Limiter(app, key_func=get_remote_address)

@app.route('/api/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    # same code
```

Without rate limiting on a login endpoint, an attacker can attempt millions of password combinations automatically. With rate limiting, they get five attempts per minute — at which point brute force becomes practically impossible.

---

### Hardcoded API Keys and Secrets

The most common security failure in AI-generated code. AI frequently places API keys, database URLs, and passwords directly in the code — because that is the simplest way to make the code run, and AI optimizes for making code run.

```python
# AI slop — key is hardcoded and will be committed to git
openai_client = OpenAI(api_key="sk-abc123yourrealkey")

# Correct — key comes from environment
import os
from dotenv import load_dotenv
load_dotenv()
openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
```

The hardcoded version works identically in development. It is catastrophically different in security — the moment it is committed to a public repository, automated scanners harvest the key within minutes.

---

### Missing Authentication and Authorization

AI builds features. Authentication checks whether a user is logged in. Authorization checks whether a logged-in user is allowed to do what they are trying to do. AI frequently builds the feature without the checks.

```python
# AI slop — no authentication check
@app.route('/api/user/<user_id>/data')
def get_user_data(user_id):
    user = User.query.get(user_id)
    return jsonify(user.sensitive_data)

# Any request to /api/user/1/data returns user 1's sensitive data.
# No login required. No ownership check.

# Correct version
@app.route('/api/user/<user_id>/data')
@login_required
def get_user_data(user_id):
    if current_user.id != int(user_id) and not current_user.is_admin:
        return jsonify({'error': 'Unauthorized'}), 403
    user = User.query.get(user_id)
    return jsonify(user.sensitive_data)
```

---

### Insecure Webhooks — No Signature Verification

A webhook is a URL on your server that an external service calls when something happens. Payment processors, GitHub, Stripe, Twilio — they all use webhooks. When a payment completes, Stripe sends a POST request to your webhook URL.

The problem: anyone can send a POST request to a URL. Without signature verification, your application cannot tell the difference between a real Stripe notification and a fake one crafted by an attacker.

```python
# AI slop — accepts any POST request as valid
@app.route('/webhook/payment', methods=['POST'])
def payment_webhook():
    data = request.json
    if data['type'] == 'payment_succeeded':
        fulfill_order(data['order_id'])  # Anyone can trigger this
    return '', 200

# Correct — verify the signature Stripe includes in the request
import stripe
@app.route('/webhook/payment', methods=['POST'])
def payment_webhook():
    payload = request.data
    sig_header = request.headers.get('Stripe-Signature')
    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, os.getenv('STRIPE_WEBHOOK_SECRET')
        )
    except ValueError:
        return '', 400  # Invalid payload
    except stripe.error.SignatureVerificationError:
        return '', 400  # Invalid signature
    if event['type'] == 'payment_intent.succeeded':
        fulfill_order(event['data']['object']['metadata']['order_id'])
    return '', 200
```

---

### No Input Validation

Every place a user can give your application data is an attack surface. Names, emails, search queries, file uploads, URL parameters — all of it needs to be validated before it touches your database or your logic.

AI generates code that handles expected input correctly. It rarely handles unexpected input at all.

```python
# AI slop — no validation
@app.route('/api/search')
def search():
    query = request.args.get('q')
    results = db.execute(f"SELECT * FROM products WHERE name LIKE '%{query}%'")
    return jsonify(results)

# The query parameter is used directly in SQL — classic injection.
# Also: no length check, no sanitization, no rate limiting.

# Correct
from markupsafe import escape
@app.route('/api/search')
@limiter.limit("30 per minute")
def search():
    query = request.args.get('q', '').strip()
    if not query or len(query) > 100:
        return jsonify({'error': 'Invalid query'}), 400
    results = db.execute(
        "SELECT * FROM products WHERE name LIKE ?",
        (f'%{escape(query)}%',)
    )
    return jsonify(results)
```

---

## What grep Is and How to Use It

Before we get to the tools, you need to know how to search your own code. `grep` is the command-line tool for searching text patterns inside files. If you have never used it, here is everything you need.

**ELI5:** grep is like `Ctrl+F` but for your entire project — all files, all folders, at once. Instead of searching one file, you search everything.

**Basic usage:**

```bash
# Search for a word in a single file
grep "api_key" config.py

# Search in all files in current directory
grep "api_key" *

# Search recursively through all folders
grep -r "api_key" .

# Search case-insensitively
grep -ri "api_key" .

# Show line numbers in results
grep -rn "api_key" .

# Search for multiple patterns at once
grep -rE "api_key|password|secret|token" .
```

**What to grep for before committing AI code:**

```bash
# Look for hardcoded credentials
grep -rn "api_key\s*=\s*['\"]" .
grep -rn "password\s*=\s*['\"]" .
grep -rn "secret\s*=\s*['\"]" .
grep -rn "token\s*=\s*['\"]" .

# Look for verify=False (SSL bypass)
grep -rn "verify=False" .

# Look for missing timeouts in requests
grep -rn "requests\.get\|requests\.post" . | grep -v "timeout"

# Look for SQL string concatenation (injection risk)
grep -rn "f\"SELECT\|f'SELECT\|% SELECT\|+ SELECT" .

# Look for hardcoded URLs that should be environment variables
grep -rn "http://\|https://" . --include="*.py" | grep -v "test\|#"

# Search git history for sensitive strings
git log --all -p | grep -i "api_key\|password\|secret" | head -50
```

---

## The Tools — What They Do, Cost, and How Hard to Set Up

These tools automate what your eyes miss. Each one catches a different category of problem. Used together, they form a review pipeline that runs before any code reaches your repository.

---

**Ruff**
What it does: Lints and formats Python code. Replaces flake8, isort, pyupgrade, and several others in a single tool. Catches style violations, unused imports, undefined variables, and hundreds of other common issues.
ELI5 analogy: A spell checker and grammar checker for code — it flags the obvious mistakes before anyone else sees them.
Cost: Free, open source.
Difficulty: Easy. One install, one command.

```bash
# macOS / Linux:
pip install ruff --break-system-packages

# Windows:
pip install ruff
ruff check .        # find issues
ruff check . --fix  # fix automatically where possible
ruff format .       # format code
```

---

**Black**
What it does: Formats Python code to a single, consistent style. No configuration, no debates. It makes every decision about spacing, line length, and formatting and applies it uniformly.
ELI5 analogy: A strict editor who reformats every document to match a house style — you do not get to argue about it.
Cost: Free, open source.
Difficulty: Easy.

```bash
# macOS / Linux:
pip install black --break-system-packages

# Windows:
pip install black

black .
```

Why this matters for security: Consistent formatting makes code easier to review. Code that is easier to review gets reviewed more carefully. Security vulnerabilities hide in complexity and inconsistency.

---

**Pylint**
What it does: Deep static analysis of Python code. Catches logic errors, type mismatches, undefined variables, unreachable code, and design problems that simpler linters miss.
ELI5 analogy: A senior developer reading your code and leaving detailed notes about everything they would question or change.
Cost: Free, open source.
Difficulty: Medium. Has many configuration options; start with defaults.

```bash
# macOS / Linux:
pip install pylint --break-system-packages

# Windows:
pip install pylint

pylint yourmodule.py
```

---

**Flake8**
What it does: Classic Python linter. Checks style (PEP 8), common errors, and complexity. Slower and less comprehensive than Ruff but widely used and well understood.
ELI5 analogy: A style guide enforcer — makes sure your code follows the agreed-upon rules.
Cost: Free, open source.
Difficulty: Easy.

```bash
# macOS / Linux:
pip install flake8 --break-system-packages

# Windows:
pip install flake8

flake8 .
```

---

**Bandit**
What it does: Security-specific static analysis for Python. Scans for known vulnerability patterns — hardcoded passwords, SQL injection, use of weak cryptography, insecure use of subprocess, and dozens more.
ELI5 analogy: A security auditor who reads your code specifically looking for the things that get people hacked — not style issues, not performance issues, just security problems.
Cost: Free, open source.
Difficulty: Easy.

```bash
# macOS / Linux:
pip install bandit --break-system-packages

# Windows:
pip install bandit

bandit -r .                    # scan everything
bandit -r . -ll                # only show medium and high severity
bandit -r . -f json -o report.json  # output as JSON report
```

This is the tool that would catch `verify=False`, hardcoded credentials, SQL string concatenation, use of `eval()`, and weak random number generation. Run it on every AI-generated codebase.

---

**MyPy**
What it does: Static type checking for Python. Catches type errors — places where a function expects a string but might receive None, or expects a number but receives a list.
ELI5 analogy: A fact-checker who verifies that every variable is the type you said it would be, before the code runs.
Cost: Free, open source.
Difficulty: Medium. Requires type annotations to be most effective; can be added gradually.

```bash
# macOS / Linux:
pip install mypy --break-system-packages

# Windows:
pip install mypy

mypy yourmodule.py
```

Why this matters for security: Type errors are a major source of unexpected behavior. Unexpected behavior is exploitable behavior.

---

**SonarQube**
What it does: Enterprise-grade code quality and security analysis platform. Scans for bugs, vulnerabilities, code smells, and security hotspots across multiple languages. Integrates with CI/CD pipelines and tracks issues over time.
ELI5 analogy: A full security and quality audit firm — not just catching individual issues but tracking the health of the entire codebase over time.
Cost: Community edition free (self-hosted). SonarCloud free for public repos, paid for private.
Difficulty: Hard. Requires setup and configuration; significant overhead for small projects. Worth it for anything production-grade with a team.

---

**CopyLeaks**
What it does: AI content detection and plagiarism checking. Identifies AI-generated content, copied code, and intellectual property issues.
ELI5 analogy: A detector that tells you how much of what you are shipping was written by AI versus a human — and whether any of it is copied from somewhere else.
Cost: Paid. Free trial available.
Difficulty: Easy to use; moderate cost consideration.

---

**pre-commit**
What it does: Framework for managing and running all of the above tools automatically before every `git commit`. If any tool fails, the commit is blocked.
ELI5 analogy: A checklist that runs itself. Every time you try to commit, pre-commit runs every tool you have configured. If anything fails, it stops the commit and tells you why.
Cost: Free, open source.
Difficulty: Easy to set up, powerful once configured.

---

## The Fix — Tools and Commands

This is the section to jump to if you want the practical setup without reading everything above.

**Install everything:**

```bash
# macOS / Linux:
pip install ruff black pylint bandit mypy pre-commit detect-secrets --break-system-packages

# Windows:
pip install ruff black pylint bandit mypy pre-commit detect-secrets
```

**Create your .pre-commit-config.yaml:**

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.4
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/psf/black
    rev: 24.4.2
    hooks:
      - id: black

  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.8
    hooks:
      - id: bandit
        args: ["-ll"]

  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-merge-conflict
      - id: debug-statements
```

**Install the hooks:**

```bash
pre-commit install
```

From this point forward, every `git commit` automatically runs ruff, black, bandit, and detect-secrets. If any of them fail, the commit is blocked and you see exactly what failed and why.

**Run manually against all files:**

```bash
pre-commit run --all-files
```

**The review checklist for every AI-generated block:**

```
Before accepting any AI-generated code:

□ Do I understand what every line does?
   If no — ask the AI to explain it line by line.

□ Are there hardcoded values that should be in .env?
   grep -rn "=\s*['\"][A-Za-z0-9]{20,}" .

□ Are there requests without timeouts?
   grep -rn "requests\.\(get\|post\|put\|delete\)" . | grep -v timeout

□ Is user input validated before use?
   Find every place request data is accessed and trace it forward.

□ Are there SQL queries built with string formatting?
   grep -rn "f\".*SELECT\|f'.*SELECT\|%.*SELECT" .

□ Is verify=False anywhere?
   grep -rn "verify=False" .

□ Are there authentication checks on every endpoint?
   Check every route decorator.

□ Is there rate limiting on any endpoint that accepts 
   user input or authenticates users?

□ Run bandit:
   bandit -r . -ll

□ Run detect-secrets:
   detect-secrets scan --all-files
```

**When to throw AI code away entirely:**

```
→ You have asked for the same security fix three times
  and the AI keeps reintroducing the vulnerability

→ The code is so long and tangled that you cannot
  identify where the security-critical logic is

→ The AI is generating code that uses libraries or APIs
  that do not exist or do not work as described

→ You have fixed five bugs and each fix created two more

→ The code handles authentication, payments, or personal
  data and you do not understand how it works
```

On security-critical code, writing it yourself with full understanding is better than shipping AI code you are guessing about.

---

## Open Source Tools — The Danger Nobody Talks About

Every open source tool you use to build your project is code you did not write, running with your permissions, on your machine, potentially with access to your credentials and your users' data.

Most open source tools are safe. Some are not. And the line between safe and unsafe can change overnight when a maintainer's account is compromised, a malicious PR is merged, or a well-intentioned project is acquired by someone with different intentions.

**How to vet an open source tool before using it:**

```
□ When was the last commit? 
  More than 12 months ago = potentially unmaintained.
  Unmaintained = unpatched vulnerabilities.

□ How many contributors does it have?
  One-person projects are higher risk — single point of failure.

□ Does it have a SECURITY.md or vulnerability reporting process?
  If not, how would you know if it had a critical CVE?

□ How many open issues are there and how old are they?
  1,000 open issues with none addressed in 6 months = abandoned.

□ Has it been recently transferred to a new owner?
  Account takeovers and malicious acquisitions happen.
  A trusted package under new ownership is worth auditing.

□ Does it have tests?
  Code without tests is harder to audit and easier to break.

□ Check pip-audit before installing:
  pip-audit --dry-run package-name
```

**Copying code from Stack Overflow, GitHub, or AI:**

```
→ Check the license. Some licenses require attribution.
  Some prohibit commercial use. Know what you are agreeing to.

→ Read the code. Do not copy what you do not understand.

→ Check the date. A Stack Overflow answer from 2015 may 
  use an approach that has since been superseded by a 
  more secure pattern.

→ AI-generated code on GitHub is not peer-reviewed code.
  A repository with AI-generated code labeled as such 
  has the same security properties as if you generated it.
```

---

## The Bottom Line

AI is an extraordinarily powerful development tool. It is not a replacement for understanding what you are building, why it is built the way it is, and what could go wrong with it.

The tools in this section exist because humans miss things. The review discipline exists because AI misses different things. Together — human review, automated tooling, and a security-first mindset — they catch what either one alone would not.

Ship fast. Ship secure. Know the difference between the two.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
