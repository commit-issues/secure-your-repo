# Credential & Secret Management

> Your credentials are the keys to everything you have built. Treat them like it.

---

!!! abstract "TL;DR"
    - Most credential leaks are not hacks — they are accidents, reused passwords, and social engineering.
    - Never hardcode credentials in code. Ever. For any reason. Even temporarily.
    - Use a .env file for secrets. Never commit it. Always have a .env.example.
    - Use a password manager. Use passphrases, not passwords. Never reuse.
    - MFA is not optional. Passkeys and hardware keys are the strongest options available.
    - Install detect-secrets and pre-commit before you write any code that touches credentials.
    - Rotate keys on a schedule — not just when something goes wrong.
    - If a credential is exposed — revoke it immediately, then clean up. Order matters.

    Experienced? Jump to [The Emergency Procedure →](#the-emergency-procedure)

---

Credentials are the number one attack vector in software security. Not because attackers are impossibly sophisticated. Because credentials are everywhere, they are reused, they are hardcoded, they are committed to git, they are pasted into Slack messages, they are written on sticky notes, they are left in screenshots, and they are handled carelessly every single day by developers who know better and do it anyway.

The goal of this section is not to make your project impenetrable. That does not exist. The goal is to make yourself difficult enough that attackers move on to easier targets. Security is not a wall — it is a cost-benefit calculation for the attacker. Make the cost high enough and they will find someone else. Do not be low-hanging fruit.

---

## How Credentials Actually Get Stolen

Before you can protect credentials, you need to understand how they actually get compromised — because most of it is not what people expect.

**Social engineering and phishing**

The most effective attack against credentials is not technical at all. It is convincing someone to hand them over. A fake GitHub security alert. A spoofed email from "your cloud provider" asking you to verify your account. A message in Discord from someone pretending to be a collaborator asking for access. A login page that looks exactly like the real one but is not.

Social engineering works because it bypasses every technical control. Your firewall, your encryption, your MFA — none of it matters if someone tricks you into typing your password into a fake login page and handing over the code from your authenticator app.

The defense: slow down. Verify independently. No legitimate service will ever ask for your credentials through an unexpected message. If something creates urgency around your login or your keys — treat it as suspicious by default.

This is exactly how account hijacking and account ransom works — they need you to panic and act fast before you think. Here is a real scenario: you get an email saying your Instagram account was logged out and you need to click here to secure it. The email looks real. The page it takes you to looks identical to Instagram's login page. You enter your credentials. Your account is now gone.

What actually happened: the email did not come from Instagram with a capital I. It came from Lnstagram — that is a lowercase L, not an I. To the eye in most fonts, they look identical. The page was a perfect clone of Instagram's login — built in minutes using AI to copy the exact design — with one small change: the form submits your credentials to the attacker's server instead of Instagram's.

Attackers use invisible Unicode characters, look-alike letters from other alphabets, upside-down punctuation, and AI-generated page clones to make fakes indistinguishable from the real thing at a glance. Your bank's login page, your email provider, your GitHub — all of it can be replicated convincingly.

The rule: never click a link in an email or message and then enter credentials on the page it takes you to. Instead, open your browser and navigate to the service directly, or open the app on your phone. Check notifications from within the app itself — not from push notifications, which can also be spoofed. If you get a suspicious email claiming to be from your bank, call the number on the back of your card. Not the number in the email.

**What to do if you get login attempt notifications:**

If you start receiving notifications that someone is attempting to log into your account — do not panic. What you are seeing is usually password cracking: an automated tool trying thousands of password combinations and failing. They have not gotten in. Your password is holding.

What to do:

```
→ Do NOT click any links in the notification emails
→ Open the service directly in your browser or app
→ Check your actual login history for any successful 
  logins you do not recognize
→ If no unauthorized access has occurred — your current 
  password is working. Consider rotating it anyway.
→ Enable MFA immediately if you have not already
→ If you see a successful login you do not recognize —
  that is a different situation. Change your password 
  immediately, revoke all active sessions, and review 
  what the attacker may have accessed.
```

**Credential stuffing and password reuse**

Every major data breach produces a list of email and password combinations that gets traded and sold on the dark web. Attackers feed these lists into automated tools that try every combination against every major service. This is credential stuffing — and it works at enormous scale because people reuse passwords.

If your password for a forum you joined in 2015 is the same as your GitHub password — and that forum was breached — your GitHub account is now in a list somewhere. The attacker does not need to hack GitHub. They just need to try the password they already have.

The defense is simple and absolute: every account gets a unique password. No exceptions. A password manager makes this practical.

**Hardcoded credentials in public repositories**

Before we go further — hardcoding is covered in its own full section later in this page, including what it means, why it happens, and how to find and fix it. This is just one of the ways it causes real damage in practice.

Automated tools scan every public commit to GitHub within minutes of it being pushed, looking for credential patterns. API keys, database URLs, tokens, private keys — all of it. If you commit a credential to a public repo, assume it has been harvested before you notice the mistake.

This happens constantly. It happens to experienced developers. It happens to security professionals. It happens because someone was moving fast, or testing something "temporarily," or forgot that a file was being tracked by git.

**Written down, screenshotted, shared in chat**

Credentials get exposed in places that seem safe in the moment. A screenshot of a terminal with an API key visible in the output. A Slack message with a token pasted to ask for help. A notes app that syncs to the cloud. A whiteboard photographed and shared.

None of these feel like security incidents when they happen. They become security incidents later.

---

## Passwords, Passphrases, and Why Length Wins

Most people think a strong password looks like this: `X7#mK!9p`. Eight characters, mixed case, numbers, symbols. Technically complex. Not actually strong by modern standards — a GPU can crack an eight-character password in hours.

A strong password actually looks like this: `correct-horse-battery-staple-47` or `riverstone.cloudmapper.driftwood.2024`. Long. Random. Memorable. These are passphrases — sequences of words that are easy to remember and computationally expensive to crack.

The math: every additional character exponentially increases the search space for a brute force attack. A 16-character passphrase of random words is more secure than an 8-character string of symbols — and infinitely more memorable.

**The rules:**

```
→ Minimum length: 16 characters
→ Better: 20-32 characters
→ High-value accounts (GitHub, email, password manager): 32+
→ Use random words — not phrases that mean something to you
→ Add numbers and symbols between words, not at the end
→ Never use the same passphrase twice
→ Store everything in a password manager —
  not your browser, not your notes app, not your memory
```

**Password managers:**

A password manager generates, stores, and fills unique passwords for every account. You remember one strong master passphrase. The manager handles everything else.

```
Recommended options:
→ Bitwarden — open source, free tier available, self-hostable
→ 1Password — strong security track record, paid
→ KeePassXC — fully local, no cloud, maximum control

Never use:
→ Your browser's built-in password manager as your primary
→ Notes apps (Apple Notes, Google Keep, Notion)
→ Spreadsheets
→ Your memory for anything you care about
```

!!! warning "Password managers are not bulletproof"
    Password managers have been breached. LastPass suffered a significant breach in 2022 where encrypted vaults were stolen. A password manager is a goldmine for attackers — if they can get your master passphrase, they have everything. Mitigate this risk:

    ```
    → Use a strong, unique master passphrase — 32+ characters
    → Enable MFA on your password manager itself
    → Choose a manager that uses zero-knowledge encryption —
      meaning even the company cannot see your passwords
    → Consider a locally stored manager (KeePassXC) if you 
      do not trust cloud storage with this level of access
    → Keep an offline backup of your most critical credentials
      in a physically secure location — not just in the manager
    ```

    A password manager is still significantly better than reusing passwords or writing them down. Use one. Just understand its limitations.

---

## Multi-Factor Authentication — The Full Picture

Most people use the terms 2FA and MFA interchangeably. They are not the same thing, and the distinction matters.

**2FA** — Two-Factor Authentication — means exactly two factors. Something you know (your password) plus something you have (your phone). Two. No more.

**MFA** — Multi-Factor Authentication — means two or more factors. It is the broader term. All 2FA is MFA, but not all MFA is 2FA. When someone says "enable 2FA," what they usually mean is "enable MFA" — use more than just a password.

**The three factor types:**

```
Something you KNOW
→ Password, PIN, passphrase, security question
→ Can be stolen, guessed, phished, or leaked
→ Weakest factor on its own

Something you HAVE  
→ Your phone, a hardware key, a backup code
→ Harder to steal remotely — requires physical access or 
  a very targeted attack
→ Can be lost, stolen, or SIM-swapped (for phone-based)

Something you ARE
→ Fingerprint, face scan, retina scan
→ Convenient and strong — you always have it
→ Cannot be rotated if compromised
→ Some systems can be fooled with photos or molds
→ Best used as one factor among several, not the only one
```

The strength of MFA comes from combining factor types. A password plus an authenticator app is two factors from two different categories — something you know and something you have. An attacker needs both. Getting one does not give them the other.

**Passkeys — the strongest option currently available**

A passkey replaces the password entirely. Instead of typing a secret, your device uses cryptography — the same concept behind SSH keys — to prove your identity. The private key never leaves your device. The server never sees it. There is nothing to phish because there is no password to steal.

Passkeys are phishing-resistant by design. A fake login page cannot capture a passkey the way it can capture a password and an SMS code. GitHub supports passkeys. Use them wherever they are available.

**Hardware security keys — YubiKey and equivalents**

A hardware security key is a physical device — roughly the size of a USB drive — that you plug in or tap to authenticate. It generates a cryptographic response that proves you have the physical device. Like a passkey but external to your computer, which means it works across devices.

```
When to use a hardware key:
→ High-value accounts — GitHub, email, cloud providers
→ Any account that, if compromised, would be catastrophic
→ When you want the strongest MFA available

YubiKey is the most widely supported option.
Google Titan Key is a solid alternative.
Both work with GitHub, Google, and most major services.
```

!!! warning "Hardware keys can be lost, stolen, or corrupted"
    A hardware key is a physical object — which means it has physical vulnerabilities. If you lose it, it is gone. If someone steals it, they have it. If it is corrupted or damaged, it stops working. This is why you should always register at least two hardware keys on any account that supports it — a primary key and a backup stored somewhere secure. Without a backup, losing your key means going through account recovery, which is slow and not always possible. Never rely on a single hardware key as your only second factor.

**Biometrics — convenient but not infallible**

Fingerprint and face authentication are strong factors because they are tied to your physical person. They are also convenient — you always have your fingerprint with you. The tradeoff: you cannot rotate your fingerprint if it is compromised. Biometrics work best as one factor in a multi-factor setup, not as the sole protection on an account.

```
Best practices for biometric authentication:

Face ID / Face unlock:
→ Enable the "require attention" or "liveness detection" 
  setting if your device offers it — this prevents someone
  from unlocking your device with a photo of your face
  while you are asleep or unaware
→ Check your device settings to confirm this is on —
  it is sometimes off by default

Fingerprint:
→ Register more than one fingerprint in your device settings
  In case of injury to one finger, you can still authenticate
→ Most devices allow 5+ fingerprints — register both index 
  fingers and at least one thumb
→ Review and remove old fingerprints if you have changed 
  devices or set them up years ago
```

**A note on phone numbers and SMS**

Using your real personal phone number for SMS-based verification creates several risks beyond SIM swapping:

```
→ SIM swapping — an attacker convinces your carrier to 
  transfer your number to their SIM. Harder than it sounds,
  but it happens, especially to people with public profiles.
  Once they have your number, they receive your SMS codes.

→ Phone lost or stolen — if your phone is gone and SMS is 
  your only second factor, you may be locked out of accounts
  until you get a new number or contact support.

→ Phone dies or is upgraded — switching to a new phone 
  without first transferring your authenticator apps means
  your 2FA codes are gone. Many people discover this at the 
  worst possible moment.

→ Number tied to your real identity — your carrier knows 
  who you are. A VOIP number is not tied to your name or 
  physical address.
```

Consider using a secondary VOIP number — a free internet-based phone number — for account verification instead of your real number. Google Voice, TextNow, and similar services provide numbers that are not tied to your physical SIM, cannot be SIM-swapped in the traditional sense, and can be accessed from any device with internet connection.

If you do use an authenticator app and get a new phone, transfer your authenticator app before you wipe or sell the old device. Most apps have an export or migration feature — use it. If you have already lost access and have no backup codes, your only option is account recovery through the service's support process — which can be slow, difficult, and not guaranteed to succeed.

```
Alternatives when you are locked out with no backup:
→ Recovery codes (if you saved them — this is why you save them)
→ Backup email or phone number on the account
→ Account recovery through support — slow but possible
→ Trusted device approval if another device is still logged in
→ For some services — identity verification with government ID
```

**Approving from trusted devices**

Some services let you approve new logins from a device you have already authenticated. A notification appears on your phone — "Did you just sign in from a new device?" — and you approve or deny it. This is a strong second factor because it requires access to your physical device and your attention.

**Backup codes — mandatory, not optional**

Every time you enable MFA on an account, you are given backup codes — one-time use codes that let you get back in if you lose your authenticator. These must be saved. Offline. Immediately.

```
→ Download them the moment they are offered
→ Store them offline — printed and secured, or in an 
  encrypted password manager
→ Never store them in email, cloud notes, or anywhere 
  that could be accessed if your account is compromised
→ Treat each code like a physical key to that account
```

**The honest truth about MFA**

Yes — determined attackers can still get through MFA. SIM swapping bypasses SMS codes. Sophisticated phishing kits can capture authenticator codes in real time. Malware on your device can intercept everything. Nothing is unbreakable.

The goal is not to make access impossible. The goal is to make it expensive enough that attackers move to easier targets. An account with a unique 32-character passphrase, a hardware security key, and backup codes stored offline is not impossible to compromise — but it is not low-hanging fruit either. Make them work for it.

---

## What Is an Environment Variable and Why Use One

If you have never heard the term "environment variable" before, this section is for you.

!!! tip "What is an environment variable?"
    Imagine your application is a chef and your API key is a secret recipe. You would not write the secret recipe on the menu for every customer to see. You keep it in a private notebook in the kitchen. The chef knows to look in the notebook when they need it.

    An environment variable is that private notebook. It is a value stored outside your code — in your operating system or a special file — that your application reads when it runs. The value never appears in your source code. It never gets committed to git. It just gets loaded when the application starts.

Your code asks: "What is the API key?" The environment answers: "Here it is." The key itself never lives in the code.

**The .env file:**

A `.env` file is a plain text file that lives in your project directory and contains your environment variables. It looks like this:

```
# .env — never commit this file
API_KEY=sk-abc123yourrealkey
DATABASE_URL=postgres://user:password@localhost/mydb
SECRET_KEY=a-very-long-random-string-here
STRIPE_SECRET=sk_live_abc123
```

This file is loaded by your application at startup. The values become available as environment variables. The file itself never gets committed to git — it is in your `.gitignore`.

**The .env.example file:**

The `.env.example` file is the public version — it shows the structure without the real values:

```
# .env.example — always commit this file
# Copy to .env and fill in your real values
API_KEY=your-api-key-here
DATABASE_URL=your-database-url-here
SECRET_KEY=choose-a-strong-random-value
STRIPE_SECRET=your-stripe-secret-key
```

This file gets committed to git. It tells anyone cloning your repo exactly what environment variables they need to set up, without exposing any real values.

**How to load .env in your code:**

```python
# Python — using python-dotenv
from dotenv import load_dotenv
import os

load_dotenv()  # loads .env file automatically

api_key = os.getenv("API_KEY")
database_url = os.getenv("DATABASE_URL")
```

```javascript
// Node.js — using dotenv
require('dotenv').config();

const apiKey = process.env.API_KEY;
const databaseUrl = process.env.DATABASE_URL;
```

**chmod — understanding and setting file permissions:**

On macOS and Linux, every file has a set of permissions that controls exactly who can read it, write to it, or execute it. These permissions are enforced by the operating system — not by your application. Even if your code never exposes a file, if the permissions are too open, another user or process on the same system can read it directly.

`chmod` is the terminal command that changes file permissions. You run it in your terminal in the directory where the file lives:

```bash
chmod 600 .env
```

**Understanding the numbers:**

chmod uses a three-digit number where each digit represents a different group:

```
First digit  → the file OWNER (you)
Second digit → your GROUP (other users in the same group)
Third digit  → EVERYONE ELSE on the system

Each digit is built by adding permission values:
4 = read
2 = write
1 = execute
0 = no permission

Examples:
7 = read + write + execute (4+2+1) — full access
6 = read + write (4+2)
4 = read only
0 = no access at all

chmod 600 means:
  6 → owner can read and write
  0 → group gets nothing
  0 → everyone else gets nothing
```

**Common permission values:**

```
chmod 600 .env        → private file, owner read/write only
chmod 700 .ssh/       → private directory, owner only
chmod 644 README.md   → public read, owner can write
chmod 755 scripts/    → public read/execute, owner can write
```

**Apply and verify:**

```bash
# macOS / Linux:
chmod 600 .env

# Verify:
ls -la .env
# Should show: -rw------- 1 youruser staff ...
# -rw------- breaks down as:
# - = regular file
# rw- = owner: read + write
# --- = group: no access
# --- = everyone else: no access
```

```
Windows equivalent:
Right-click .env → Properties → Security tab
→ Click Edit
→ Remove all users except your own account
→ Set your account to Read and Write only
→ Click Apply → OK
```

Run `chmod 600 .env` every time you create a new .env file. Verify with `ls -la .env` after creation and after any restore from backup.

!!! warning "If you accidentally commit your .env file"
    Stop what you are doing. Do not just delete the file and commit again — that does not remove it from git history. Follow the emergency procedure at the bottom of this section. Every value in that file must be considered compromised and revoked immediately.

---

## What Is Hardcoding and Why It Is Dangerous

Hardcoding means placing any value directly in your source code that should instead come from a configuration file, environment variable, or secure storage. It is not just API keys and passwords — it is any piece of information that should not be visible to anyone who reads the code.

PII stands for Personally Identifiable Information — any data that can be used to identify a real person. Names, email addresses, phone numbers, usernames, IP addresses, physical addresses, account IDs. When PII is hardcoded into source code and that code is committed to a public repository, every person whose information appears in that code has been exposed — without their knowledge or consent.

Beyond PII and credentials, hardcoding creates a broader problem: anything hardcoded becomes part of your attack surface. A hardcoded email address tells an attacker what account to target. A hardcoded username tells them what identity to impersonate. A hardcoded internal URL tells them what systems exist inside your infrastructure. Attackers do not just look for API keys — they read code looking for any information that helps them understand your environment, your accounts, and your weaknesses.

!!! tip "What does hardcoding mean?"
    Imagine tattooing your house key onto your arm and then going swimming at a public pool. Everyone there can see it. Hardcoding sensitive information in your code is the same thing — the information is right there in the source, visible to anyone who reads the file, anyone who clones the repo, anyone who finds an old commit in the history, and anyone who gets access to the codebase through a supply chain attack or compromised dependency.

```python
# Hardcoded — dangerous
import openai
client = openai.OpenAI(api_key="sk-abc123yourrealkey")

# Correct — loaded from environment
import openai
import os
from dotenv import load_dotenv

load_dotenv()
client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))
```

The hardcoded version works identically in development. The moment it is committed to a public repository, the key is exposed. The correct version works the same way but the key lives in `.env` and never touches git.

**Where hardcoded values hide in AI-generated code:**

AI consistently hardcodes values because it is optimizing for "makes it run" not "makes it secure." Look for these patterns:

```bash
# Find hardcoded credentials with grep:

# Strings that look like API keys (long alphanumeric values)
grep -rn "=\s*['\"][A-Za-z0-9_\-]{20,}['\"]" . --include="*.py"

# Common credential variable names with values
grep -rn "api_key\s*=\s*['\"]" . --include="*.py"
grep -rn "password\s*=\s*['\"]" . --include="*.py"
grep -rn "secret\s*=\s*['\"]" . --include="*.py"
grep -rn "token\s*=\s*['\"]" . --include="*.py"

# Database connection strings
grep -rn "postgresql://\|mysql://\|mongodb://" . --include="*.py"
```

**Finding hardcoded values in your codebase:**

This is a separate but connected step — once you know what hardcoding looks like, use these commands to find it across your entire project. Run these before every commit on any AI-generated codebase.

```bash
# Find strings that look like API keys — long random alphanumeric values
grep -rn "=\s*['"][A-Za-z0-9_\-]{20,}['"]" . --include="*.py"

# Find common credential variable names with hardcoded values
grep -rn "api_key\s*=\s*['"]" . --include="*.py"
grep -rn "password\s*=\s*['"]" . --include="*.py"
grep -rn "secret\s*=\s*['"]" . --include="*.py"
grep -rn "token\s*=\s*['"]" . --include="*.py"
grep -rn "email\s*=\s*['"]" . --include="*.py"
grep -rn "username\s*=\s*['"]" . --include="*.py"

# Find database connection strings
grep -rn "postgresql://\|mysql://\|mongodb://" . --include="*.py"

# Find internal URLs that should be environment variables
grep -rn "http://\|https://" . --include="*.py" | grep -v "test\|#\|example"

# Run across all file types, not just Python
grep -rn "api_key\|password\|secret\|token" . --include="*.js" --include="*.ts" --include="*.env.example"
```

Note: these grep commands find candidates — you still need to read each result and determine whether it is hardcoded or loaded from an environment variable. A line like `api_key = os.getenv("API_KEY")` will appear in results but is correctly handled.

---

## Temp Files — Always Remove Them

Temporary files are one of the most overlooked attack vectors in development. They are created constantly — debugging output, test data, API responses saved for inspection, downloaded files, generated reports, cached responses — and they accumulate quietly in your project directory. Most developers forget they exist until something goes wrong.

The reason they are dangerous: they often contain exactly the kind of sensitive data attackers are looking for, and they are almost never added to .gitignore because nobody thinks about them when setting up a project. Make cleaning them up a habit — right after you are done with them, not before the next commit.

**How attackers exploit forgotten temp files:**

A temp file containing credentials or user data that gets committed — even once, even briefly — follows the same rules as any other committed secret. It is in the history. Deleting it does not remove it. Automated scanners harvest it. And because temp files are generated by your application, they often contain data more sensitive than your source code: real API responses with real tokens, real database query results with real user data, real debug output with environment variables printed to the console.

Even on a private repository, temp files sitting in your project directory can be accidentally committed by a `git add .` that was not reviewed carefully. They can be included in zip archives sent to collaborators. They can appear in screenshots. They can be accessed by other processes on the same machine.

```
Common temp files that contain sensitive data:
→ debug.log — often contains printed environment variables,
  stack traces with file paths, and internal system details
→ test_output.json — may contain real API responses with tokens
→ downloaded_data.csv — may contain real user PII from the database
→ cache/ directories — may contain authenticated API responses
  that include session tokens or user data
→ *.tmp files — generated by editors, build tools, and scripts,
  often forgotten entirely
→ response_debug.txt, output.json, test.db — one-off debugging
  files that never get deleted
```

**Add these to your .gitignore immediately:**

```
# Temp files
*.tmp
*.temp
*.log
debug/
cache/
tmp/
temp/
test_output*
downloaded_*
response_*
output_*
*.bak
```

**Find and clean temp files before every commit:**

```bash
# Find temp files in your project
find . -name "*.tmp" -o -name "*.log" -o -name "*.temp" 2>/dev/null | grep -v ".git"

# Find untracked files not in .gitignore
git status --short | grep "^?"

# Review what git is tracking that should not be there
git ls-files | grep -E "\.(log|tmp|temp|bak)$"
```

!!! warning "If a temp file was already committed"
    Adding it to .gitignore stops future updates — but the committed version is still in your history. If it contained sensitive data, follow the history rewrite procedure in [Git History Auditing](../hardening/history.md).

---

## What Is pre-commit and How It Works

!!! tip "What is a pre-commit hook?"
    Every time you run `git commit`, git can automatically run a script first. If the script finds a problem — like a credential in your staged files — it blocks the commit and tells you what it found. You fix the problem, then commit successfully. The hook runs automatically. You do not have to remember to check.

    Think of it like a security guard at the door of your repository. Every time you try to commit, the guard checks what you are bringing in. Credentials? Not allowed. Clean code? Come on through.

**Installing pre-commit:**

```bash
# macOS:
pip3 install pre-commit detect-secrets --break-system-packages

# Linux:
pip install pre-commit detect-secrets --break-system-packages

# Windows:
pip install pre-commit detect-secrets
```

**Creating the baseline:**

Before the hook can detect new secrets, it needs to know what already exists in your codebase — otherwise it would block every commit with a false alarm about things you already know about.

```bash
detect-secrets scan > .secrets.baseline
```

This creates a snapshot of everything that currently looks like a secret. The hook will only alert you about new things that appear after this baseline.

**Creating .pre-commit-config.yaml:**

```yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: check-added-large-files
      - id: detect-private-key
      - id: check-merge-conflict
      - id: trailing-whitespace
      - id: end-of-file-fixer
```

**Installing the hook:**

```bash
pre-commit install
```

From this point forward, every `git commit` automatically scans staged files. If it finds something, it blocks the commit and shows you exactly what triggered it.

**If the hook blocks a legitimate commit:**

```bash
# Review what triggered it
detect-secrets scan --baseline .secrets.baseline

# If it is a false positive — update the baseline
detect-secrets scan > .secrets.baseline
git add .secrets.baseline
```

!!! warning "Never use --no-verify to bypass the hook without reading what it found"
    The hook blocked you for a reason. Read it. If it is a real credential — fix it before committing. If it is a false positive — update the baseline properly. Bypassing without reading is how credentials get committed.

---

## Key Rotation Schedule

Rotating credentials means replacing them with new ones on a regular schedule — not just when something goes wrong. Rotation limits the window of exposure if a key was silently compromised without your knowledge.

```
Every 90 days — routine rotation:
→ Generate new key at the source service
→ Update .env with the new value
→ Test that everything still works
→ Revoke the old key
→ Document the rotation date

Immediate rotation required when:
→ Any credential was committed to git — even briefly
→ Any credential appeared in a log file
→ Any credential was shared in a chat or screenshot
→ A collaborator who had access leaves the project
→ Any system that had access to credentials is compromised
→ You are unsure whether a credential was exposed
```

---

## GitHub Secrets and CI/CD Credentials

When your CI/CD pipeline needs credentials — to deploy, to run tests against a real API, to push to a registry — those credentials cannot live in your workflow files. They go in GitHub Secrets.

```
Settings → Secrets and variables → Actions → New repository secret
```

!!! tip "What is CI/CD?"
    CI/CD stands for Continuous Integration / Continuous Deployment. It is the automated pipeline that runs your tests, builds your project, and deploys it to a server every time you push code. GitHub Actions is GitHub's built-in CI/CD platform. We covered this in detail in [Vibe Coding & AI Dev →](vibe-coding.md#actions-settings--what-is-cicd-and-why-does-it-matter-here). The short version: it is code that runs automatically on GitHub's servers, and it often needs credentials to do its job — deploy keys, API keys, service account tokens. Those credentials must be stored securely, not in your workflow files.

**Types of secrets:**

```
Repository secrets — available to all workflows in this repo
Environment secrets — available only to specific deployment environments
  (staging, production) — use these for production credentials
Organization secrets — available across multiple repos in an org
```

**How to use secrets in a workflow:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: ./deploy.sh
```

The secret value is injected at runtime. It never appears in your workflow file. It is masked in logs automatically.

**How secrets still get exposed in logs:**

GitHub masks known secret values in logs — but only if the value exactly matches what is stored. If your workflow accidentally prints a derived value, a partial value, or processes the secret in a way that transforms it before printing — it may not be masked.

```yaml
# This will expose your secret in logs
- run: echo "Connecting to $DATABASE_URL"

# This will not
- run: |
    if [ -z "$DATABASE_URL" ]; then
      echo "DATABASE_URL is not set"
      exit 1
    fi
    ./connect.sh
```

Never echo, print, or log secret values in workflow steps. Never use secret values in step names or job names — those appear in the Actions UI.

---

## Specific Credential Types

**API keys**

```
→ Request only the scopes you need — not full access
→ Use separate keys for development and production
→ Set expiration dates where the service supports it
→ Rotate every 90 days minimum
→ Revoke immediately if exposed
→ Store in .env, load via os.getenv()
```

**Database credentials**

```
→ Never use the same credentials in development and production
→ Create a dedicated database user for your application
  with only the permissions it needs
→ Never use the root/admin database user in application code
→ Rotate after any team member leaves
→ Store connection strings in .env, never in code
```

**Tokens (OAuth, JWT, API tokens)**

```
→ Set the shortest expiration that still works for your use case
→ Request only the scopes you actually need
→ Revoke immediately when no longer needed
→ Never store tokens in localStorage in browser apps —
  use httpOnly cookies instead
→ Implement token refresh — do not ask users to re-authenticate
  every time a token expires
```

**Encryption keys**

```
→ The most critical credential in any encrypted system
→ If lost — encrypted data may be permanently unrecoverable
→ Store a backup offline in a physically secure location
→ Never store in the same place as the encrypted data
→ Never store in git — not even encrypted versions of the key
→ Rotate requires re-encrypting all data — plan for this
```

---

## Masking and Redacting Credentials

Masking means displaying a credential in a way that reveals enough to identify it without revealing the full value. Redacting means removing sensitive information entirely before it appears somewhere it should not be.

Both matter in more situations than most developers realize.

**In logs — never log full credential values:**

```python
# Wrong — full key in the log
import logging
logging.info(f"Connecting with API key: {api_key}")

# Correct — masked, shows enough to identify which key
masked = api_key[:4] + "****" + api_key[-4:]
logging.info(f"Connecting with API key: {masked}")

# For database URLs — mask the password portion
import re
safe_url = re.sub(r"://([^:]+):([^@]+)@", r"://:****@", database_url)
logging.info(f"Connecting to: {safe_url}")
```

**The envcheck pattern — verifying without exposing:**

When you need to confirm environment variables are set without displaying their full values:

```bash
# Add to ~/.zshrc or ~/.bashrc
alias envcheck='python3 -c "
import os
from dotenv import load_dotenv
load_dotenv()
keys = ["API_KEY", "DATABASE_URL", "SECRET_KEY"]
for key in keys:
    val = os.getenv(key)
    if val:
        masked = val[:4] + chr(42)*4 + val[-4:] if len(val) > 8 else chr(42)*8
        print(f"{key}={masked}")
    else:
        print(f"{key}=NOT SET")
"'
```

This lets you verify your environment is configured correctly without credentials appearing in your terminal output — or your terminal history.

**Before sharing screens or screenshots:**

```
→ Close terminal tabs with .env contents visible
→ Check your editor for open files containing credentials
→ Blur or crop sensitive portions before sharing screenshots
→ Never paste credential values into chat, Discord, or Slack
→ If you accidentally shared a credential — revoke immediately
```

**In CI/CD logs:**

```yaml
# Never do this
- run: echo "Key is ${{ secrets.API_KEY }}"

# Verify a secret is set without revealing it
- run: |
    if [ -z "$API_KEY" ]; then
      echo "API_KEY is not set"
      exit 1
    fi
    echo "API_KEY is set (${#API_KEY} characters)"
```

**In .env.example files:**

```
# Wrong — looks like it could be real
API_KEY=sk-abc123

# Correct — clearly a placeholder
API_KEY=your-api-key-here
```

Never use values in .env.example that look like real credentials. Anyone who sees a realistic-looking key may not realize it is a placeholder.

---

## The Emergency Procedure

A credential has been exposed. This is the order of operations — do not skip steps, do not reorder them.

```
Step 1 — Confirm and then act.
  Before revoking, take 30 seconds to verify this is real.
  Is the credential actually exposed, or is this a false alarm?
  Check: is it in a public commit? In a log? In a screenshot?
  Confirm the exposure is real — then act immediately.
  A false alarm handled with speed causes disruption.
  A real exposure handled slowly causes a breach.
  Once confirmed: go to the service that issued the credential
  and invalidate it now. Revocation stops the bleeding.

Step 2 — Generate a new credential.
  Do this immediately after revoking.
  Do not leave a gap where the service has no valid credential.

Step 3 — Update your .env with the new value.
  Test that everything works with the new credential.

Step 4 — Rewrite git history if the credential was committed.
  Use git filter-repo to remove it from every commit.
  Full procedure: [Git History Auditing →](../hardening/history.md)

Step 5 — Force push the clean history.
  git push origin main --force

Step 6 — Request a GitHub cache purge.
  https://support.github.com
  Even after rewriting, GitHub caches old commit views.

Step 7 — Audit for unauthorized use.
  Check usage logs at the service where the credential was issued.
  Look for API calls, logins, or actions you did not perform.
  Check your GitHub security log for unexpected activity.

Step 8 — Notify if necessary.
  If the credential gave access to user data, payment systems,
  or any third-party service — follow that service's incident
  response process. You may have legal notification obligations.
```

!!! warning "Speed matters — but so does verification"
    Automated scanners harvest credentials from public repositories within minutes of a push. Confirm the exposure is real, then act without delay. Every minute a live credential sits exposed is a minute it can be used against you. Confirm first. Revoke second. Clean up third. Investigate fourth.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
