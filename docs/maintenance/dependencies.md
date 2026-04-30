# Dependency Intelligence

Installing a dependency is a one-time decision. Trusting it is ongoing.

You vetted it when you installed it. You checked the download count, the maintainer history, the last commit date — or you should have. It passed. You moved on. But the threat landscape didn't stop moving when you did. A package that was clean on install day may have a critical CVE published against it tomorrow. The maintainer account may get compromised next week. The project may be abandoned quietly next month — no announcement, no final patch, just silence while vulnerabilities accumulate.

This section is about what happens after the install. The setup-time decisions are covered in [Dependency Security](../code/dependencies.md). This is the ongoing intelligence layer that keeps you informed after that.

---

## What Changes After Install

**New CVEs get published against existing packages — and that can change at any moment.** A vulnerability discovered today applies to every version of that package already installed in every project that uses it. Your `requirements.txt` doesn't update itself. Your `package.json` doesn't send you a notification. That's exactly why tools like the [CVE Security Intelligence Monitor](https://github.com/commit-issues/cve-security-monitor) exist — and why setting up automated cron jobs like we covered in [Cron & Scheduled Tasks](../code/cron.md) matters. The CVE gets published, the exploit code follows, and your project stays exposed until you patch it.

**Maintainer accounts get compromised.** A package that was trustworthy under its original maintainer may publish a malicious version after an account takeover. The package name is the same. The install command is the same. New versions come out and those new versions come with changes — changes you didn't authorize and didn't expect. See [Supply Chain Security](../threats/supply-chain.md) for how this plays out in practice.

**Packages get abandoned.** An unmaintained package doesn't get security patches. It doesn't get CVE fixes. It just sits in your dependency tree, getting older, while the vulnerabilities that get discovered against it go unaddressed. No announcement. No warning. Just a package that stopped being safe while you weren't watching.

**Transitive dependencies change.** The packages your packages depend on can update independently. A direct dependency you've pinned and vetted may pull in a transitive dependency that changed since you last looked.

---

## Your Ongoing Monitoring Stack

These are the tools that watch your dependencies so you don't have to do it manually every day.

### Dependabot

If you followed [Dependency Security](../code/dependencies.md) and [Advanced Security](../hardening/advanced-security.md), Dependabot is already configured. It watches your dependency graph continuously and opens PRs when it finds vulnerabilities or outdated packages.

The maintenance task is staying on top of the alert queue — not letting it grow into a backlog you never review. An ignored Dependabot alert is security debt. See [Security Debt](debt.md).

```
GitHub → Repo → Security tab → Dependabot alerts
```

For each open alert: triage it, patch it, or document why you're deferring it. Never dismiss without a reason.

### pip-audit and npm audit on a Schedule

Running audits manually is better than not running them. Running them automatically is better than manually. Wire them into your cron schedule so they run regularly without you having to remember.

=== "macOS / Linux / Kali"
    ```bash
    # Add to crontab — weekly audit every Monday at 8am
    0 8 * * 1 /usr/bin/python3 -m pip_audit >> /home/youruser/logs/pip-audit.log 2>&1
    0 8 * * 1 cd /home/youruser/project && /usr/bin/npm audit >> /home/youruser/logs/npm-audit.log 2>&1
    ```

=== "Windows (WSL2)"
    ```bash
    # Same commands inside WSL2 crontab
    0 8 * * 1 /usr/bin/python3 -m pip_audit >> /home/youruser/logs/pip-audit.log 2>&1
    ```

See [Cron & Scheduled Tasks](../code/cron.md) for how to set this up securely with proper logging.

### CVE Security Intelligence Monitor

If you want ongoing dependency monitoring built into a tool that also tracks CVEs, security news, and sends alerts — the [CVE Security Intelligence Monitor](https://github.com/commit-issues/cve-security-monitor) (`commit-issues/cve-security-monitor`) is an open source tool built for exactly this.

Here's how it works: the monitor runs on a schedule, pulls from 17 curated security intelligence sources — NVD, CISA, threat feeds, security news outlets — and cross-references what it finds against your environment. When a new CVE drops that matches something you're running, you find out. Not when you remember to check. Not when Dependabot eventually opens a PR. When it happens. It stores everything in an encrypted local database, exports to CSV, JSON, or PDF, and sends alerts via desktop notification or email depending on how you configure it. It's designed to run quietly in the background and only surface when something actually matters — no noise, just signal.

It runs `pip-audit` as part of its quality gate before every commit, and the upcoming version will include OSV Scanner as an additional scanning layer — two scanners, one dependency tree, maximum coverage. If you're building something that monitors security for others, your own dependency hygiene needs to be exemplary. This tool is built to that standard.

### OSV Scanner

If you want a standalone scanner to run alongside your existing tools — or you're not using the CVE monitor — OSV Scanner is the next best addition to your stack. It's free, open source, and maintained by Google's Project Zero team. It queries the [Open Source Vulnerabilities database](https://osv.dev) — which aggregates CVEs from GitHub Advisory, PyPI Advisory, npm Advisory, and more — and catches vulnerabilities that `pip-audit` and `npm audit` sometimes miss.

Run it alongside your existing tools, not instead of them. Multiple scanners catch different things.

=== "macOS / Linux / Kali"
    ```bash
    # Install OSV Scanner
    brew install osv-scanner  # macOS
    # or
    curl -L https://github.com/google/osv-scanner/releases/latest/download/osv-scanner_linux_amd64 -o osv-scanner
    chmod +x osv-scanner
    sudo mv osv-scanner /usr/local/bin/

    # Scan your project
    osv-scanner --lockfile requirements.txt
    osv-scanner --lockfile package-lock.json

    # Scan recursively
    osv-scanner -r /home/youruser/project/
    ```

=== "Windows (WSL2)"
    ```bash
    # Download the Linux binary inside WSL2
    curl -L https://github.com/google/osv-scanner/releases/latest/download/osv-scanner_linux_amd64 -o osv-scanner
    chmod +x osv-scanner
    sudo mv osv-scanner /usr/local/bin/

    # Scan your project
    osv-scanner --lockfile requirements.txt
    osv-scanner --lockfile package-lock.json
    ```

=== "Windows (Native)"
    ```powershell
    # Download the Windows binary from:
    # https://github.com/google/osv-scanner/releases/latest
    # Look for: osv-scanner_windows_amd64.exe

    # Run it
    .\osv-scanner_windows_amd64.exe --lockfile requirements.txt
    ```

---

## When an Alert Fires

An alert is not an emergency by default. It is information that requires a decision.

**Triage first.** Not every CVE is exploitable in your specific context. A vulnerability in a package you use only for local development tooling is different from one in a package that handles user input in production. Read the CVE. Understand the attack vector. Decide whether it's actually exploitable in your use case.

**Patch when you can.** If there's a fixed version available and upgrading doesn't break your project, upgrade. Run your tests. Verify nothing broke. Commit with a clear message.

```bash
# Python
pip install --upgrade package-name
pip-audit  # verify clean

# Node
npm update package-name
npm audit  # verify clean
```

**Document when you defer.** If you can't patch immediately — the fix introduces a breaking change, you're mid-release, the upgrade needs more testing — write down why and set a date. An undocumented deferral is invisible debt.

**Remove if you can't fix.** If a package is abandoned, has no fix available, and poses genuine risk — find an alternative or remove the dependency entirely. A package you don't have is a package that can't be exploited.

---

## Abandonment Signals — What to Watch For

These are signs a dependency may be becoming a liability:

```
□ No commits in over 12 months with open security issues
□ Maintainer account shows no recent activity
□ Issues and PRs piling up with no responses
□ Package marked as deprecated on the registry
□ Download counts dropping sharply
□ No response to reported vulnerabilities
```

When you see these signals, start evaluating alternatives before you're forced to switch under pressure.

---

## Dependency Intelligence Checklist

```
□ Dependabot alerts reviewed regularly — none dismissed without reason
□ pip-audit / npm audit running on a schedule
□ OSV Scanner added to your scanning stack
□ Transitive dependencies reviewed, not just direct
□ Abandoned packages identified and alternatives evaluated
□ Alert triage process defined — patch, defer with reason, or remove
□ Audit logs reviewed — scanner output not just ignored
□ Lock files committed and up to date
```

---

## Where This Goes Next

One section left. Your notification system is the last surface — how you stay informed without turning your alerting setup into a credential leak.

[→ Notifications](notifications.md) — How notification configuration becomes a security surface and how to stay informed without leaking.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
