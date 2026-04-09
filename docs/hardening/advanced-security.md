# Advanced Security

> GitHub ships every repo with a security suite most people never configure. Free. Passive. Always running. The gap between a secured repo and an exposed one is often just these settings — and the five minutes it takes to turn them on.

---

!!! abstract "TL;DR"
    - Enable the dependency graph first — everything else depends on it.
    - Dependabot alerts catch known CVEs. Malware alerts catch something worse — actively malicious packages.
    - Push Protection is the most important setting on this page. Never disable it.
    - For serious projects — enable CodeQL, pip-audit in CI, and dependency review on every PR.
    - Schedule automated scans. Do not rely on manual checks.
    - A healthy security posture has zero unresolved critical alerts and PRs reviewed within 48 hours.
    - Write your SECURITY.md. It tells researchers how to reach you before they go public.

    Auditing an existing repo? Jump to [What a Healthy Security Posture Looks Like](#what-a-healthy-security-posture-looks-like)

---

There is a version of security that is reactive. Something goes wrong, you find out about it, you fix it. This is how most people operate — and it works, until it doesn't. Until the breach already happened. Until the malicious package already ran. Until the exposed secret was already harvested. By the time you are reacting, the damage is done.

The settings in this section are the other version. Preventative. Automated. Always running, always watching, catching problems before they become incidents. GitHub provides all of this for free on public repositories. It requires no maintenance once configured. And yet the majority of public repos on GitHub have none of it enabled.

This might feel like overkill if you are building a small personal tool. It is not. Here is why.

The cost of enabling every setting in this section is exactly zero — zero dollars, zero ongoing effort, zero performance impact on your project. The cost of not having them, when something goes wrong, is not zero. A compromised dependency in a healthcare tool that handles patient data is not a minor incident. A malicious package in a financial application is not something you recover from quietly. A leaked API key in a production system that processes payments is not a debugging session — it is a breach notification, a legal conversation, and potentially a regulatory fine.

Better to be preventative and paranoid than to f*ck around and find out.

Even if you are building "just a tool" today — tools grow. They get forked. They get used in ways you did not anticipate. The person who set it up right from day one never has to have the conversation about why they didn't. Set it up right. Now.

```
Settings → Advanced Security
```

---

## Start Here: The Dependency Graph

Before you enable anything else, enable the dependency graph. Every other Dependabot feature depends on it — literally. Without the dependency graph, GitHub cannot see what your project depends on, and therefore cannot alert you when those dependencies have problems.

```
✅ Dependency graph — Enable first
```

The dependency graph maps every package your project depends on — direct dependencies and their dependencies, all the way down. For a Python project it reads your `requirements.txt`. For a Node project it reads `package.json`. For a project with a `Dockerfile` it maps those dependencies too. The result is a complete picture of everything running in your project that you did not write yourself.

**Automatic dependency submission** goes one step further. For compiled languages like Java, Go, and Swift, the final dependency list is only known after the build runs — not from the manifest file alone. Automatic dependency submission hooks into your build process and reports the full resolved dependency tree, including build-time dependencies that would otherwise be invisible.

```
✅ Automatic dependency submission — Enable if you use 
   compiled languages (Java, Go, Swift, Kotlin, Scala)
   Leave disabled for Python/Node/Ruby — not needed
```

Once the dependency graph is enabled, everything else in this section becomes possible.

---

## Dependabot Alerts

```
✅ Dependabot alerts — Always enabled
```

Dependabot alerts notify you when a dependency in your project has a known security vulnerability — a CVE (Common Vulnerabilities and Exposures) that has been published to the National Vulnerability Database or GitHub's own advisory database.

The way it works: GitHub continuously monitors the dependency graph you just enabled against a database of known vulnerabilities. The moment a CVE is published for a package you depend on — regardless of when you installed that package — GitHub sends you an alert. You do not have to check. You do not have to run a scan. It happens automatically, around the clock, for every dependency in your project.

This is exactly the kind of passive protection that most developers wish they had set up after something goes wrong. The dependency that was fine six months ago when you installed it may have a critical CVE published against it today. Without Dependabot alerts, you find out when someone exploits it. With them, you find out when the CVE is published.

Under Dependabot alerts you will see **Dependabot rules** — this lets you create custom rules for how alerts are handled. The default rule (Dismiss low impact alerts for development-scoped dependencies) is reasonable. Leave it unless you have a specific reason to change it.

---

## Dependabot Malware Alerts

```
✅ Dependabot malware alerts — Always enabled, no exceptions
```

This is a separate toggle from regular Dependabot alerts and it protects against something categorically different — and worse.

A regular Dependabot alert means: "This package has a known vulnerability. It may be exploitable." The package itself is legitimate. It has a flaw. You patch it.

A malware alert means: "This package is a weapon." It was created or modified with the specific intent to harm systems that install it. It is not a vulnerable package — it is an attack. The distinction matters enormously because the response is completely different.

When you receive a Dependabot CVE alert, you review it, assess the severity, and merge the fix PR when ready. When you receive a malware alert, you ***stop everything***. You remove the package immediately. You audit what it had access to while it was installed. You check whether it made any outbound connections, accessed any credentials, or wrote any files it should not have. You treat it as a potential breach, not a maintenance task.

Malware in the npm and PyPI ecosystems is a documented and growing problem. Attackers publish packages with names nearly identical to popular ones — a single transposed character, a hyphen vs underscore — and wait for developers to accidentally install them. They compromise the accounts of legitimate package maintainers and push malicious updates to packages with millions of downloads. They submit pull requests to popular projects that add their package as a dependency, betting that maintainers will not scrutinize every new dependency added by a contributor.

This toggle makes GitHub watch for all of it. There is no reason not to have it on.

---

## Dependabot Security Updates

```
✅ Dependabot security updates — Enable
```

Alerts tell you there is a problem. Security updates do something about it.

When Dependabot detects a vulnerability in one of your dependencies, security updates automatically opens a pull request with the fix — updating the vulnerable package to a patched version, with a clear description of the vulnerability and why this version resolves it. You review the PR, run your tests, and merge it when you are ready.

This transforms security maintenance from something you have to remember to do into something that happens automatically. The alert fires, the PR appears, you review and merge. No manual `pip install --upgrade`, no hunting through changelogs, no remembering to check six months from now.

!!! warning "Dependabot PRs still go through your review process"
    Dependabot PRs are not auto-merged. They follow the same branch protection rules you configured in [Branch Protection →](rulesets.md) — which means they require your review and must pass your status checks before anything merges. This is intentional. A supply chain attack that compromises a package and publishes a "security update" would arrive as a Dependabot PR — your review is the last line of defense. Read the diff. Check what changed. Then merge.

---

## Grouped Security Updates

```
✅ Grouped security updates — Enable
```

If your project has many dependencies, Dependabot can generate a lot of PRs in a short period — especially when a batch of CVEs drops at once or when you have not merged updates in a while. Grouped security updates bundles related fixes into a single PR instead of one PR per vulnerability.

This keeps your PR queue manageable and reduces the review overhead for routine maintenance. Instead of reviewing twelve individual Dependabot PRs, you review one PR that addresses all twelve. The security coverage is identical — the operational experience is significantly cleaner.

---

## Dependabot Version Updates

```
✅ Dependabot version updates — Enable with dependabot.yml
```

Security updates only fire when there is a known CVE. Version updates go further — they keep your dependencies current even when there is no vulnerability, simply because staying current reduces the gap between where you are and where a future vulnerability might hit you.

A dependency pinned to a version from eighteen months ago has eighteen months of potential CVEs between it and the current release. A dependency kept current has days. The attack surface is smaller, the patches are smaller, and the review burden per update is smaller.

Version updates require a configuration file. Create `.github/dependabot.yml` in your repository:

**For a Python project:**

```yaml
version: 2
updates:
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    reviewers:
      - "yourusername"
    labels:
      - "dependencies"
      - "security"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
    reviewers:
      - "yourusername"
    labels:
      - "dependencies"
      - "github-actions"
```

**For a Node project, add:**

```yaml
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 10
```

**For a project with Docker:**

```yaml
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "daily"
```

A few things worth noting about this configuration. The `github-actions` ecosystem entry keeps your workflow Actions updated — including pinning new SHA values when Actions publish new versions. This is the automated version of what we covered in [GitHub Settings →](settings.md#actions-permissions). The `reviewers` field assigns you as the reviewer on every Dependabot PR so nothing slips through unreviewed. The `labels` field tags PRs for easy filtering in your PR queue.

For a project that handles sensitive data, processes payments, or operates in a regulated environment — set the interval to `daily`. For a personal project with fewer dependencies — `weekly` is reasonable. The difference in notification frequency is worth it for anything where a vulnerability sitting unpatched for a week is an unacceptable risk.

!!! tip "Dependabot version updates on a docs repo"
    Even for a documentation site like this one, keeping GitHub Actions updated via Dependabot is valuable. A stale Actions version is a potential supply chain vector — and Dependabot handles the SHA update automatically, which means you never have to manually hunt for new SHAs. Enable the `github-actions` ecosystem entry regardless of what kind of project you are running.

---

## Code Scanning — CodeQL

```
For personal/docs projects:  ☐ CodeQL — optional
For code projects:           ✅ CodeQL — strongly recommended  
For sensitive/regulated:     ✅ CodeQL — non-negotiable
```

CodeQL is GitHub's static analysis engine. It reads your source code and looks for security vulnerabilities — SQL injection patterns, authentication bypasses, data exposure paths, insecure deserialization, and dozens of other vulnerability classes that are easy to miss in manual review. It does not run your code. It analyzes the code itself, looking for patterns that match known vulnerability signatures.

For a documentation site, CodeQL has little to scan. Skip it or leave it disabled — it will not cause harm but it will not provide meaningful value either.

For any project that handles user input, connects to a database, processes payments, stores personal data, or exposes an API — enable CodeQL. Set it up on every push to main and every pull request. A vulnerability class that CodeQL would have caught is not a defense in a breach post-mortem.

To enable CodeQL, click "Set up" next to CodeQL analysis. GitHub will offer a default setup or an advanced setup. For most projects, default setup is sufficient — it automatically detects your languages and configures appropriate queries. Advanced setup gives you a custom workflow file if you need to tune which queries run.

```
✅ Copilot Autofix — OFF
```

Copilot Autofix suggests AI-generated fixes for CodeQL alerts. Keep it off. The principle here is the same one that runs through this entire guide — you review your own code. An AI-generated fix for a security vulnerability needs the same scrutiny as any other code change, possibly more. Read the alert, understand the vulnerability, write or review the fix yourself. Do not let automation make security decisions for you.

---

## Automated Security Scanning in CI

The settings above run passively on your repository. For a serious project — especially one with contributors, frequent pushes, or sensitive functionality — you also want active scanning built into your CI/CD pipeline so every push and every PR is scanned automatically.

Add these to your GitHub Actions workflow:

**pip-audit on every push:**

```yaml
name: Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * *'  # Daily at 6am UTC

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      
      - name: Set up Python
        uses: actions/setup-python@0b93645e9fea7318ecaed2b359559ac225c90a2b
      
      - name: Install dependencies
        run: pip install -r requirements.txt --break-system-packages
      
      - name: Run pip-audit
        run: |
          pip install pip-audit --break-system-packages
          pip-audit --requirement requirements.txt
      
      - name: Run detect-secrets
        run: |
          pip install detect-secrets --break-system-packages
          detect-secrets scan --baseline .secrets.baseline
```

**Dependency review on pull requests:**

```yaml
name: Dependency Review

on:
  pull_request:
    branches: [main]

jobs:
  dependency-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      
      - name: Dependency Review
        uses: actions/dependency-review-action@3b139cfc5fae8b618d3eae3675e383bb1769c019
```

The dependency review action blocks PRs that introduce packages with known vulnerabilities before they even get to the merge stage. It is the automated enforcement version of "check your dependencies before you add them."

**How often to run scans:**

```
Solo project, no contributors:
→ On every push to main
→ Daily scheduled scan

Project with contributors:
→ On every push to main
→ On every pull request — especially from external contributors
→ Daily scheduled scan

Project handling sensitive data or with many dependencies:
→ On every push to main
→ On every pull request, required to pass before merge
→ Daily scheduled scan minimum
→ Immediately when a critical CVE drops for any dependency
   (Dependabot handles this automatically)
```

The scheduled scan (`cron`) is the one most people skip. It catches vulnerabilities in dependencies that were safe when you last pushed but have had CVEs published since then — without requiring you to push anything new to trigger a scan.

---

## Secret Protection and Push Protection

```
✅ Secret Protection — Always on for public repos
✅ Push Protection — Never disable this
```

Secret Protection scans your repository for credentials matching hundreds of known patterns — API keys, tokens, connection strings, private keys. GitHub maintains partnerships with major service providers — AWS, Google Cloud, Stripe, Twilio, and dozens more — so when a matching pattern is detected, GitHub not only alerts you but also notifies the service provider, who can automatically revoke the exposed credential.

Push Protection is the more important of the two because it acts before exposure rather than after. When you attempt to push a commit containing a recognized secret pattern, GitHub blocks the push and tells you exactly which file and line contains the secret. The credential never reaches GitHub. It never becomes part of your public history. It is stopped at the door.

***Never disable Push Protection.*** There is no legitimate reason to. If the hook is blocking a push because of a false positive — a test value, an example credential, a placeholder — you can bypass it for that specific push with a justification, and GitHub logs that bypass. The protection remains active for everything else.

!!! warning "Push Protection only catches known patterns"
    Push Protection is excellent at catching credentials it recognizes — and it recognizes a lot. But it cannot catch everything. A custom API key format your employer uses internally, a proprietary token format, a manually generated secret — these may not match any known pattern. This is why detect-secrets in your pre-commit hook and your CI pipeline is a complementary layer, not a redundant one. Push Protection is the last line of defense, not the only one.

---

## Protection Rules Thresholds

At the bottom of the Code Scanning section you will find the Protection rules thresholds. These control at what severity level a code scanning alert will block a PR from merging.

```
Security alert severity: High or higher  ← leave this
Standard alert severity: Only errors     ← leave this
```

High or higher means Critical and High severity CodeQL findings block merges. Medium and Low findings generate alerts but do not block. This is the right balance for most projects — it stops the serious issues while not creating alert fatigue from every minor finding.

For a banking, healthcare, or otherwise regulated project — consider moving the threshold to Medium or higher. In a regulated environment, a Medium severity finding that makes it to production is a compliance conversation you do not want to have.

---

## SECURITY.md — Telling Researchers How to Reach You

Private vulnerability reporting, which we enabled in [GitHub Settings →](settings.md), is the mechanism. SECURITY.md is the signpost that tells researchers the mechanism exists and how to use it.

GitHub looks for a `SECURITY.md` file in your repository root or in your `.github/` folder. When it finds one, it displays a link to it on your repository's Security tab and on any issue or PR creation page. Security researchers who find a vulnerability in your project look for this file before they do anything else. If they do not find it, they either go public immediately or give up. Neither outcome is good for you.

Create `.github/SECURITY.md`:

```markdown
# Security Policy

## Supported Versions

| Version | Supported |
|---------|-----------|
| latest  | ✅        |
| older   | ❌        |

## Reporting a Vulnerability

If you discover a security vulnerability in this project, please report it 
privately rather than opening a public issue.

**To report a vulnerability:**

Use GitHub's private vulnerability reporting:
Repository → Security tab → "Report a vulnerability"

This ensures your report goes directly to the maintainer without being 
publicly visible while the issue is being investigated and resolved.

**What to include:**
- A description of the vulnerability
- Steps to reproduce it
- The potential impact
- Any suggested fixes if you have them

**What to expect:**
- Acknowledgment within 48 hours
- Regular updates on the investigation
- Credit in the fix disclosure if you would like it

Thank you for helping keep this project secure.
```

Adjust the response time commitment to what you can actually deliver. A 48-hour acknowledgment is reasonable for a solo project. Do not promise a 24-hour fix if you cannot deliver it — an honest slower timeline is better than a fast timeline you cannot keep.

---

## What a Healthy Security Posture Looks Like

Once Advanced Security is fully configured, your repository's Security tab becomes a live dashboard of your security posture. Here is what healthy looks like versus what needs attention.

**Healthy:**
```
✅ Dependabot alerts: 0 unresolved Critical or High
✅ Secret scanning alerts: 0 open
✅ Code scanning: last scan green, no unresolved High+ findings
✅ All Dependabot security PRs reviewed within 48 hours
✅ dependabot.yml committed and running on schedule
✅ SECURITY.md present and current
```

**Needs attention:**
```
⚠️  Dependabot alerts piling up unreviewed
⚠️  Secret scanning alert open for more than 24 hours
⚠️  CodeQL not run in more than 7 days
⚠️  Dependabot PRs sitting unmerged for weeks
⚠️  No SECURITY.md file
⚠️  dependency graph disabled
```

Security posture drifts when maintenance is deferred. A single unreviewed Dependabot alert becomes ten. Ten become twenty. At some point the queue feels too large to address and it gets ignored entirely. The drift is gradual and the consequences are not — when a critical vulnerability is exploited in a dependency that had a patch sitting in an unmerged Dependabot PR for three months, the PR queue is not the story. The breach is.

Review your Security tab at least once a week for active projects. Once a month minimum for anything with dependencies. Treat unresolved Critical alerts as urgent. Treat High alerts as this week. Treat Medium as this sprint. Treat Low as on the backlog with a date.

---

## When You Get a Notification — What to Do Next

Every alert type in this section has a different level of urgency and a different response. Knowing which is which before an alert fires means you are not making decisions under pressure when it does.

For the full response playbook — what to do when Dependabot opens a PR, when a secret scanning alert fires, when a malware alert drops, and when a CodeQL finding appears — see [Troubleshooting & Recovery →](../setup/troubleshooting.md).

The short version:

```
Dependabot CVE alert → review severity, merge fix PR within SLA
  Critical/High:  treat as urgent, same day
  Medium:         this week
  Low:            this sprint, scheduled

Dependabot malware alert → stop everything
  Remove the package immediately
  Audit what it accessed while installed
  Treat as a potential breach, not a patch

Secret scanning alert → act immediately
  Revoke the exposed credential at the source
  Generate a new one
  Rewrite history if needed
  Full procedure: Troubleshooting & Recovery →

CodeQL finding → assess and fix
  High/Critical: block merge until resolved
  Medium: fix before next release
  Low: backlog with a date

Push Protection block → read before bypassing
  The hook found something — go look at it
  If it is a false positive, bypass with justification
  If it is real — fix it, do not bypass
```

---

## The Full Settings Reference

```
DEPENDENCY GRAPH
✅ Dependency graph                  → Always enabled
✅ Automatic dependency submission   → Enable for compiled languages

DEPENDABOT
✅ Dependabot alerts                 → Always enabled
✅ Dependabot malware alerts         → Always enabled, no exceptions
✅ Dependabot security updates       → Always enabled
✅ Grouped security updates          → Always enabled
✅ Dependabot version updates        → Enable with dependabot.yml

CODE SCANNING
☐  CodeQL (docs/simple projects)    → Optional
✅ CodeQL (code projects)            → Strongly recommended
✅ CodeQL (sensitive/regulated)      → Non-negotiable
☐  Copilot Autofix                  → Always disabled

PROTECTION RULES
✅ Security alert severity           → High or higher (Medium+ for regulated)
✅ Standard alert severity           → Only errors

SECRET PROTECTION
✅ Secret Protection                 → Always on (cannot disable on public repos)
✅ Push Protection                   → Never disable
```

---

Next: [Day One Checklist →](checklist.md)

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
