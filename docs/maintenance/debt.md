# Security Debt

Security debt is like a bill you didn't know was coming. You thought you were covered. You thought that thing you skipped was fine, that the token you didn't rotate wasn't a big deal, that the permission you left open wasn't worth the time to fix. Then it arrives — and it's never just the original amount. It comes with compounding interest. The longer it sat, the more it cost. And unlike a credit card statement, it doesn't send you a reminder before it's due.

The sections below are that reminder.

---

## What Security Debt Actually Is

Security debt is the gap between the security posture your project should have and the one it actually has. It accumulates the same way technical debt does — through shortcuts taken under pressure, decisions deferred because something else was more urgent, and configurations that were fine once but have since become liabilities.

The difference between security debt and technical debt is consequence. Messy code slows you down. Unpatched security debt gets you breached.

It shows up in three forms:

**Deferred maintenance.** The things you knew needed doing and didn't do. A Dependabot alert you dismissed. An outdated Action you've been meaning to pin. A collaborator whose access you meant to revoke when they left.

**Configuration drift.** Settings that were correct when you made them but have drifted out of alignment as your project changed. A workflow with permissions scoped to features you no longer use. A deploy key for a server you decommissioned. A branch protection rule that no longer reflects how your team works.

**Accumulated exposure.** Decisions that seemed acceptable at the time but stack up over time. Using the same token across multiple services. Never rotating SSH keys. Never checking what's in your git history. Each one is a small bet. Enough small bets and the odds catch up.

---

## How It Accumulates

Security debt almost always starts with shipping pressure. The feature has to go out. The deadline is real. The security thing can wait — it's not broken, it's just not optimal.

That's usually true. Until it isn't.

> ⚠️ **This is especially true in business**
> Companies will push products and features out the door as fast as possible to generate revenue — while being painfully slow to invest in security tooling, hire qualified professionals, or pay for proper audits. The logic is that security doesn't make money. The reality is that a breach, a lawsuit, a compliance failure, or a forced shutdown costs orders of magnitude more than the security investment they avoided. The bill always comes. Businesses just tend to get the largest one.

The problem isn't that any single deferred item is catastrophic. The problem is the compounding. A dependency goes unpatched for three months. A CVE is published against it in month two. An exploit is in the wild by month three. The window between "this could have been patched" and "this is an active incident" closes fast — and the longer the debt sits, the more it costs to pay.

The other accumulation pattern is turnover. A project changes hands, grows a team, brings on contractors. Access gets added and rarely gets removed. Workflows get copied from other projects without review. The original security decisions get buried under layers of changes made by people who didn't make them.

---

## How to Audit What You Owe

This is a one-time sweep to find what's accumulated. Run it against every active repo.

### Dependencies

```bash
# Python
pip-audit
pip3-audit

# Node
npm audit

# Review outdated packages
pip list --outdated
pip3 list --outdated
npm outdated
```

Flag anything with a known CVE. Flag anything that hasn't been updated in over a year with no clear reason why.

### Access Controls

```
GitHub → Repo → Settings → Collaborators and teams
```

For every collaborator: Do they still need access? Is their access level still appropriate? If you can't answer yes to both, act on it.

```
GitHub → Settings → Developer settings → Personal access tokens
GitHub → Repo → Settings → Deploy keys
GitHub → Settings → SSH and GPG keys
```

For every token and key: What is it for? Is it still in use? When was it last used? If you don't know, revoke it.

### GitHub Actions

```bash
# Find Actions pinned to tags instead of SHAs
grep -r "uses:" .github/workflows/ | grep -v "@[a-f0-9]\{40\}"
```

Every result is a mutable reference — something that can change without your knowledge. Pin to SHAs. See [Advanced Security](../hardening/advanced-security.md).

Review permissions on every workflow:

```yaml
# What you should see at the top of every workflow
permissions:
  contents: read
  # Only the permissions the job actually needs
```

If you don't see explicit permissions, the workflow is running with defaults that are broader than necessary.

### Git History

```bash
# Check for exposed emails
git log --format="%ae" | sort -u

# Check for exposed names
git log --format="%an" | sort -u

# Search for anything that looks like a credential
git log -p | grep -iE "(api_key|secret|password|token|credential)" | head -20
```

Anything that surfaces here is already public if your repo is public. See [OSINT & Identity Leakage](../threats/osint.md#fixing-what-youve-already-leaked) for the remediation process.

### Open Security Alerts

```
GitHub → Repo → Security tab
```

Review every open Dependabot alert, every secret scanning alert, and every code scanning result. For each one: is there a reason this is still open? A dismissed alert without a documented reason is debt.

---

## How to Pay It Down

Don't try to fix everything at once. Prioritization is what makes debt payable.

**Critical — fix immediately:**
- Known CVEs with public exploits in your dependencies
- Exposed credentials in git history or live files
- Active tokens or keys you cannot account for
- Open secret scanning alerts

**High — fix this week:**
- Dependabot alerts without a documented reason for deferral
- Collaborators with access that should have been revoked
- Actions pinned to tags instead of SHAs
- Workflows with overly broad permissions

**Medium — fix this month:**
- Outdated dependencies without known CVEs
- SSH keys and deploy keys that haven't been rotated recently
- Git history with personal information
- Missing or outdated `SECURITY.md`

**Low — fix next cycle:**
- Documentation that doesn't reflect current security decisions
- Configuration that works but could be tightened
- Processes that rely on manual steps that should be automated

> 💡 **Small consistent payments beat one big overhaul**
> Set aside time each week — even thirty minutes — specifically for security debt. A dependency updated, a token rotated, a permission tightened. It compounds in the right direction over time.

---

## Preventing New Debt

The best time to address security is before it becomes debt. A few habits that stop it from accumulating:

**Make security part of your definition of done.** Before a feature ships, run the audit commands. Check what you added. Review what changed. Five minutes before the commit is worth days after the breach.

**Document your decisions.** When you dismiss a Dependabot alert, write down why. When you grant a permission, note what it's for. When you defer something, set a date. Undocumented decisions become invisible debt.

**Review access when people leave.** The moment a collaborator, contractor, or teammate is no longer active on your project, revoke their access. Not eventually. Immediately.

**Run the freshness check regularly.** See [Data Freshness](freshness.md) for the full cadence. The freshness check is your early warning system — it surfaces debt before it has time to compound.

**Use your checklist.** The [Security Checklist](../hardening/checklist.md) exists precisely for this. Each item checked is debt that doesn't get created.

---

## Security Debt Checklist

```
□ Dependency audit run — all CVEs triaged
□ Open Dependabot alerts reviewed and either fixed or documented
□ Collaborator access audited — no ghost access
□ Tokens and keys audited — nothing unaccounted for
□ GitHub Actions pinned to SHAs
□ Workflow permissions scoped to minimum required
□ Git history scanned for credentials and personal info
□ Security tab reviewed — no unaddressed alerts
□ Dismissed alerts documented with reason and date
□ Access revoked for anyone no longer active on the project
```

---

## Where This Goes Next

Debt is the past catching up with you. The next section is about the present — specifically, the security risks hiding inside your scheduled jobs and automated workflows.

[→ Cron & Automation](cron.md) — The security risks of scheduled jobs, what they can expose, and how to run them safely.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
