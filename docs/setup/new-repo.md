# Starting a New Repository Right

> The decisions you make in the first five minutes of a new repo protect everything that comes after.

---

!!! abstract "TL;DR"
    - Create the repo on GitHub with intention — visibility, license, and README set on day one.
    - Create your .gitignore ***before*** your first commit. Adding it after does not remove secrets already pushed.
    - Set up your .env file and .env.example before writing any code that touches credentials.
    - Install detect-secrets and configure the pre-commit hook so secrets are caught before they ever reach GitHub.
    - Verify your git config — email, signing key, and author name — before committing anything.
    - Run through the first commit checklist before you push. Every time.
    - If you already pushed a secret — stop, revoke it at the source, then fix the history.

    Already past your first commit? Jump to [If You Already Exposed a Secret](#if-you-already-exposed-a-secret)

---

Every secure repository starts before the first line of code. The setup decisions you make — or skip — in those first few minutes become the foundation everything else is built on. A .gitignore added after an accidental push, credentials discovered in a commit from six months ago, a repo set to public when it should have been private — these are not edge cases. They happen constantly, to developers of every experience level, and most of them are completely preventable.

This section is the playbook. Follow it in order, every time you start something new. It takes less than fifteen minutes and it works for any project — a personal tool, a portfolio piece, a production application, or anything in between.

---

## Step 1 — Create the Repo on GitHub With Intention

Before you touch your terminal, make decisions on GitHub first. These settings are easier to get right at creation than to fix after the fact.

```
Go to: github.com → New repository

Repository name:
  → Lowercase, hyphens not underscores
  → Descriptive but concise
  → No personal info in the name

Visibility:
  → Public: Anyone can see the code. Choose this intentionally.
  → Private: Only you and collaborators can see it.
  → When in doubt — start private. You can make it public later.
    You cannot un-expose code that was already public.

Initialize with:
  → Add a README — yes, always. It creates your first clean commit.
  → Add .gitignore — skip this here. We will create a proper one manually.
  → Choose a license — MIT for open source. No license means no permissions granted to anyone.
```

!!! warning "Visibility is a one-way door — in one direction"
    Making a private repo public is instant and irreversible in one critical way — anything that was ever in the commit history is now exposed to the world, including anything you thought was safely buried. Search engines index public repos. Automated scanners harvest them within minutes of going public. ***Decide visibility before you write a single line, not after.***

!!! tip "Not sure which license to use?"
    For most open source projects, MIT is the right choice. It is simple, permissive, and widely understood. It lets people use, modify, and distribute your code as long as they keep your copyright notice. If you want more control over how your code is used commercially, look into Apache 2.0 or GPL. If you are not ready to decide — add a license placeholder and revisit it. A repo with no license is not "unlicensed open source" — it is legally closed by default.

---

## Step 2 — Clone the Repo and Set Up Git Identity

Once the repo exists on GitHub, clone it to your machine using SSH. If you have not set up SSH keys yet, do that first — [SSH Keys →](ssh.md)

```
git clone git@github-youraliasname:yourusername/your-repo-name.git
cd your-repo-name
```

Before you do anything else, confirm your git identity is correct for this repo:

```
git config user.email
git config user.name
```

These should return your GitHub noreply address and the name you want attached to commits. If they return your personal email or a name you do not want public — fix them now, before the first commit:

```
git config user.email "123456789+yourusername@users.noreply.github.com"
git config user.name "yourusername"
```

!!! tip "Global vs local git config"
    Running `git config` without `--global` sets the value for this repo only. Running it with `--global` sets it for every repo on your machine. If you use one GitHub account for everything, set it globally once. If you have multiple accounts — personal, work, etc. — set the identity at the repo level so each project uses the right one. We covered this in [SSH Keys → Configuring for Multiple Accounts](ssh.md#configuring-sshconfig-for-multiple-accounts).

Confirm commit signing is configured:

```
git config commit.gpgsign
```

This should return `true`. If it returns nothing or `false`, set it up now — [SSH Keys → Setting Up SSH Commit Signing](ssh.md#setting-up-ssh-commit-signing)

---

## Step 3 — Create Your .gitignore Before Anything Else

This is the most skipped step and the one with the most irreversible consequences. ***Your .gitignore must exist before your first real commit.***

Here is why this matters so much. Git tracks every file you commit — including secrets, database files, environment variables, and credentials. A .gitignore tells git which files to never track. But it only prevents files from being added going forward. If a secret was committed before the .gitignore existed, git already knows about it. Adding the .gitignore after the fact does not remove it from history. The secret is still there, in every clone of the repo, forever — unless you surgically rewrite history, which is painful, error-prone, and time-consuming.

***Set up the .gitignore first. Before any other files. Before any credentials exist anywhere in the project.***

Create the file:

```
macOS / Linux:
nano .gitignore

Windows:
notepad .gitignore
```

Paste this as your starting point and add anything specific to your project below it:

```
# Environment and credentials — never committed
.env
.env.production
.env.local
.env.*.local
*.key
*.pem
*.p12
*.pfx

# Database files
*.db
*.sqlite
*.sqlite3

# Backup and sensitive folders
backups/
secrets/
private/

# Python
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
*.egg-info/
dist/
build/
venv/
.venv/

# Node
node_modules/
npm-debug.log
yarn-error.log

# macOS
.DS_Store
.AppleDouble
.LSOverride

# Windows
Thumbs.db
desktop.ini

# Editor noise
.vscode/
.idea/
*.swp
*.swo
*.sublime-project
*.sublime-workspace

# MkDocs / documentation builds
site/

# Logs
*.log
logs/

# Secrets baseline
.secrets.baseline
```

Save the file. Then verify git sees it:

```
git status
```

You should see `.gitignore` listed as a new untracked file. That is correct.

!!! warning "The cautionary tale nobody forgets after it happens to them"
    A developer builds a quick prototype. They put their API key directly in the code "just for testing." They commit it to a public repo. They realize the mistake twenty minutes later and delete the file — then commit again thinking the secret is gone.

    It is not gone. Git history is permanent. The original commit with the API key is still there, still accessible, still cloneable. Within hours an automated scanner has harvested it. Within days charges appear on the developer's account from API calls they did not make.

    The fix — revoking the key and rewriting history — takes two hours. The charges take weeks to dispute. The lesson costs more than the project was worth.

    ***The .gitignore goes in first. Always.***

---

## Step 4 — Set Up Your Credentials Pattern

Before you write any code that touches API keys, database passwords, or any other sensitive value, establish the pattern you will use to handle them.

The correct pattern has two files:

```
.env          ← real values, never committed, in .gitignore
.env.example  ← placeholder values, always committed, shows structure
```

Create your `.env.example` first — this is the file that goes into the repo:

```
# .env.example — commit this file
# Copy to .env and fill in real values — never commit .env

API_KEY=your-api-key-here
DATABASE_URL=your-database-url-here
SECRET_KEY=choose-a-strong-random-value-here
```

Then create your actual `.env` with real values — this file never gets committed:

```
# .env — never commit this file
API_KEY=sk-abc123yourrealkey
DATABASE_URL=postgres://user:password@localhost/dbname
SECRET_KEY=s8f7g3h2j9k0l1m4n5
```

!!! tip "Why .env.example?"
    It does two things at once. First, it documents exactly which environment variables your project needs — anyone who clones the repo knows what to set up. Second, it does this without exposing any real values. The real values live only in your local .env file, which git ignores. This is the pattern used in virtually every serious open source project and it is the right approach from day one.

!!! warning "Never put real values in .env.example"
    It sounds obvious. It happens constantly. Someone copies their working .env into .env.example to "make it easier for contributors" and commits it. Every value is now public. If you are ever unsure whether a value in .env.example is real — it should not be there.

Load your .env values in code using a library, not by hardcoding them. In Python:

```python
from dotenv import load_dotenv
import os

load_dotenv()
api_key = os.getenv("API_KEY")
```

In Node.js:

```javascript
require('dotenv').config();
const apiKey = process.env.API_KEY;
```

The value never appears in your source code. It lives only in the .env file on each machine that runs the project.

---

## Step 5 — Install detect-secrets and Set Up the Pre-Commit Hook

A pre-commit hook is a script that runs automatically every time you run `git commit`. If the script finds a problem — like a credential pattern in your staged files — it blocks the commit before anything reaches git history.

detect-secrets is a tool that scans for high-entropy strings and known credential patterns. It is one of the best first lines of defense against accidental secret exposure.

**Install detect-secrets:**

```
macOS / Linux:
pip install detect-secrets --break-system-packages

Windows:
pip install detect-secrets
```

**Create a baseline scan of your current codebase:**

```
detect-secrets scan > .secrets.baseline
```

This creates a snapshot of any strings that look like secrets in your current files. Anything in the baseline is considered known and acknowledged — the hook will only alert you about new ones.

!!! tip "What is a baseline?"
    Think of it like a security audit snapshot. The first time you run detect-secrets it finds everything that looks suspicious. You review the list, acknowledge what is intentional (like example placeholder values), and save it as the baseline. From then on, the hook only flags things that are new — things that appeared since the last baseline scan. This prevents false alarms on things you have already reviewed.

**Install the pre-commit framework:**

```
macOS / Linux:
pip install pre-commit --break-system-packages

Windows:
pip install pre-commit
```

**Create the pre-commit config file:**

```
nano .pre-commit-config.yaml
```

Paste this:

```yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.5.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

Save and exit. Then install the hook:

```
pre-commit install
```

You should see:

```
pre-commit installed at .git/hooks/pre-commit
```

From this point forward, every time you run `git commit` the hook automatically scans your staged files. If it finds something that looks like a credential, it blocks the commit and shows you exactly which file and line triggered it.

!!! warning "The hook only protects you if you let it"
    If the hook blocks a commit and you bypass it with `--no-verify`, you are turning off your own protection. When that happens — stop and read what the hook found. It is almost always right. The one time you bypass it and it was right is the one time you will regret it.

---

## Step 6 — Your First Commit Checklist

Before you run `git commit` for the first time, go through this list. It takes two minutes and it prevents the kind of mistakes that take hours to fix.

```
Open a new terminal and run:

git status
```

Read every file listed. Ask yourself:

```
□ Is .gitignore listed as a new file and committed?
□ Is .env absent from the staged files?
□ Is .env.example present and containing only placeholder values?
□ Is .secrets.baseline committed?
□ Is .pre-commit-config.yaml committed?
□ Are there any files staged that should not be public?
□ Does git config user.email return your noreply address?
□ Does git config commit.gpgsign return true?
```

Then review the actual diff before committing:

```
git diff --staged
```

Read every line. If you see a real API key, a real password, a real email address, a database URL with credentials, or anything that should not be public — unstage that file and fix it before proceeding.

When everything passes — commit:

```
git add .gitignore .env.example .secrets.baseline .pre-commit-config.yaml README.md
git commit -S -m "init: project setup with security foundations"
git push origin main
```

!!! tip "Why stage files individually for the first commit?"
    Using `git add .` stages everything in the directory. On a first commit that is usually fine — but reading each file name forces you to consciously acknowledge what you are committing. It takes ten seconds and it catches things that `git add .` would silently include. Make it a habit for first commits especially.

---

## If You Already Exposed a Secret

If a credential has already been committed and pushed — even if you deleted the file immediately after — ***assume it is compromised***. Automated scanners index public GitHub repos continuously. The window between a push and a harvest can be minutes.

Do this immediately, in this order:

```
Step 1 — Revoke the exposed credential at the source.
  Go to wherever the API key, password, or token was issued
  and invalidate it immediately. Do not wait.

Step 2 — Generate a new credential.
  Replace it before you do anything else.

Step 3 — Update your .env with the new value.

Step 4 — Rewrite git history to remove the old value.
  Use git filter-repo — the modern, safe way to do this.

  Install git filter-repo:
  pip install git-filter-repo --break-system-packages

  Remove the file from history entirely:
  git filter-repo --path path/to/file-with-secret --invert-paths

  Or replace the specific string throughout history:
  git filter-repo --replace-text expressions.txt
  (where expressions.txt contains: old-secret-value==>REMOVED)

Step 5 — Force push the clean history:
  git push origin main --force

Step 6 — Contact GitHub support to purge cached views
  of the old commits. Go to:
  https://support.github.com

Step 7 — Audit any systems that used the exposed credential
  for unauthorized activity.
```

!!! warning "Deleting the file is not enough"
    Removing a file in a new commit does not remove it from git history. The original commit where the secret appeared is still there, still accessible, still cloneable by anyone who had a copy of the repo before your fix. ***Revocation at the source is the only real fix for a compromised credential. History rewriting removes the evidence but the credential must be considered burned the moment it was pushed.***

---

## Ongoing — Every New Repo, Every Time

Once you have done this once it becomes muscle memory. Here is the order distilled:

```
1. Create repo on GitHub — set visibility and license intentionally
2. Clone using SSH
3. Verify git identity — email and signing
4. Create .gitignore — before any other files
5. Create .env.example — before writing any credential-touching code
6. Install detect-secrets and pre-commit hook
7. Run first commit checklist
8. Commit signed, push
```

Every repo. Every time. No exceptions.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
