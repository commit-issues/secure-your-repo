# Day One Checklist

This is your cheat sheet. Everything in Part 1 and Part 2 distilled into one place.

> Already have your GitHub account and SSH set up and hardened? [Jump to Repository Setup →](#repository-setup)

---

## Profile & Account Setup

*These are done once — when you first set up your GitHub account or a new machine. You do not repeat these for every repo.*

---

**Step 1 — GitHub Account Security**
*Done once per GitHub account.*

```
□ GitHub account uses a dedicated purpose-only email address
□ That email account is also secured with 2FA
□ Passkey enabled — OR authenticator app (not SMS, not Google Auth)
□ Authenticator app supports encrypted backup
□ Backup codes downloaded and stored offline
□ Commit email set to GitHub noreply address:
    GitHub → Settings → Emails → Keep my email address private
□ Block command line pushes that expose my email → enabled
□ Vigilant Mode enabled:
    GitHub → Settings → SSH and GPG keys → Vigilant mode
□ SSH keys audited — nothing unrecognized or stale
□ Third-party app access reviewed — nothing unnecessary
□ Password is strong, unique, stored in a password manager
□ Account recovery options do not use your personal phone number
```

→ Full guide: [Securing Your GitHub Account](../setup/account.md)

---

**Step 2 — SSH Keys**
*Done once per machine.*

```
□ ed25519 SSH key generated with a strong passphrase:
    ssh-keygen -t ed25519 -C "noreply@users.noreply.github.com"
□ Key added to SSH agent
□ Public key (.pub) added to GitHub as Authentication Key
□ Public key (.pub) added to GitHub as Signing Key
□ Connection tested: ssh -T git@github.com
□ ~/.ssh/config configured if using multiple accounts
□ Commit signing configured:
    git config --global gpg.format ssh
    git config --global user.signingkey ~/.ssh/id_ed25519.pub
    git config --global commit.gpgsign true
```

→ Full guide: [SSH Keys](../setup/ssh.md)

---

## Repository Setup

*These are done for every new repository.*

---

**Step 3 — Before You Write Any Code**

```
□ Repo created with correct visibility — decided intentionally
□ License added — MIT for open source
□ Cloned via SSH using correct alias:
    git clone git@github-alias:you/repo.git
□ Git identity verified before first commit:
    git config user.email  → must be your GitHub noreply address
    git config user.name   → must be your GitHub username
    git config commit.gpgsign  → must return true
□ .gitignore created BEFORE any other files
□ .env.example created with placeholder values only
□ detect-secrets installed and baseline created:
    detect-secrets scan > .secrets.baseline
□ pre-commit hook installed:
    pre-commit install
```

→ Full guide: [Starting a New Repository](../setup/new-repo.md)

---

**Step 4 — GitHub General Settings**

```
Settings → General → Features

☐ Wikis          → Disabled (or enabled + restrict to collaborators)
☐ Issues         → Disabled if not managing contributions
☐ Sponsorships   → Disabled unless actively sought
☐ Discussions    → Disabled
☐ Projects       → Disabled for solo projects
✅ Pull requests  → Enabled, single merge strategy only
☐ Commit comments → Disabled
```

→ Full guide: [GitHub Settings](../hardening/settings.md)

!!! tip "Running an open source project with contributions or sponsorships?"
    The settings above are the baseline. Open source projects that accept contributions, sponsorships, or have a public user base require a significantly more hardened posture — contributor vetting, malicious PR protection, maintainer OPSEC, and sponsorship identity considerations. This is covered in full in [Open Source Security →](../threats/open-source.md)

---

**Step 5 — GitHub Actions Settings**

```
Settings → Code and automation → Actions → General
⚠️ Save each section separately

✅ Actions permissions → Your account + selected only
✅ Require SHA pinning → Enabled
✅ Fork PR approval   → All external contributors
✅ Workflow permissions → Read only
☐ Actions create/approve PRs → Always disabled
```

→ Full guide: [GitHub Settings](../hardening/settings.md)

---

**Step 6 — Branch Ruleset**

```
Settings → Branches → Rulesets → New branch ruleset

✅ Ruleset name: main-protection
✅ Enforcement status: Active
✅ Bypass list: Repository admin ← check this first
✅ Target: Include default branch
✅ Restrict deletions
✅ Block force pushes
✅ Require signed commits
```

→ Full guide: [Branch Protection](../hardening/rulesets.md)

---

**Step 7 — Advanced Security**

```
Settings → Advanced Security

✅ Dependency graph → Enable first
✅ Dependabot alerts → Always on
✅ Dependabot malware alerts → Always on
✅ Dependabot security updates → Always on
✅ Grouped security updates → Always on
✅ Dependabot version updates → Enable with dependabot.yml
☐ CodeQL → Optional for docs, required for code projects
☐ Copilot Autofix → Always off
✅ Secret Protection → Always on
✅ Push Protection → Never disable
```

→ Full guide: [Advanced Security](../hardening/advanced-security.md)

---

**Step 8 — Required Files**

```
□ README.md
    → States the official repo URL
    → Includes license info
    → Links to security reporting

□ LICENSE
    → MIT for open source
    → Present before first public commit

□ .github/SECURITY.md
    → Tells researchers how to report vulnerabilities
    → Sets response time expectations

□ .github/CODEOWNERS (for team projects)
    → Defines who must review changes to sensitive files

□ .github/dependabot.yml
    → Configured for your package ecosystem
    → Schedule set to daily for active projects
```

→ Templates in: [Advanced Security](../hardening/advanced-security.md)

---

**Step 9 — First Commit Audit**

```
Before git commit — run through this every time:

□ git status → only intended files staged
□ git diff --staged → read every line
□ .env is NOT in staged files
□ No credentials or tokens in the diff
□ No real email addresses in any file
□ No real names or machine names in file paths
□ .gitignore committed
□ .env.example committed with placeholders only
□ .secrets.baseline committed
□ .pre-commit-config.yaml committed

Commit signed:
git commit -S -m "init: project setup with security foundations"
git push origin main

Verify the Verified badge appears on the commit on GitHub.
```

→ Full guide: [Starting a New Repository](../setup/new-repo.md)

---

**Step 10 — History Audit**
*Run once on every new or inherited repo. Run again any time you add a new collaborator or machine.*

```
□ git log --format="%ae" | sort | uniq
  → Every email is a noreply address

□ git log --format="%an" | sort | uniq
  → No real names, no old handles

□ git log --all -p | grep -i "/Users/\|/home/\|C:\\Users\\"
  → No real usernames in file paths

□ trufflehog git file://. --only-verified
  → No secrets found in history

□ Repo → Insights → Contributors
  → Every contributing account is intentional
```

→ Full guide: [Git History Auditing](../hardening/history.md)

---

## Ongoing Maintenance

*Not every repo, not every commit — but on a schedule.*

```
Every commit:
□ Pre-commit hook runs — let it, read what it flags
□ Read your diff before committing

Every week:
□ Check Security tab for open alerts

Every month:
□ Review Dependabot alerts and PRs
□ Run pip audit manually

Every 90 days:
□ Rotate all API keys
□ Audit SSH keys — revoke anything stale
□ Review third-party app access

Every 6 months:
□ Full history audit
□ Review repo access and collaborators
□ Test your backup restore

After any incident:
□ Full audit — credentials, history, access
□ Rotate everything that could have been exposed
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
