# Logging, Auditing & Accountability

*Logs are your evidence. When something goes wrong, they are how you prove what happened, when, and from where.*

---

!!! info "TL;DR"

    - Most applications either log too much — including sensitive data they should never store — or too little — nothing useful when something goes wrong.
    - There are four distinct types of logs. They serve different purposes, have different audiences, and should be stored and retained differently.
    - Never log passwords, full API keys, session tokens, or unmasked PII. Log the event. Not the secret.
    - Audit logs are immutable. You do not edit them. You do not delete them. They are evidence.
    - GDPR and CCPA give users the right to have their data deleted — but that does not mean deleting audit evidence. It means anonymizing PII while keeping the event record.
    - In a public repo that accepts pull requests, your logging and audit system is your accountability layer for every change that has ever touched your codebase.

    Already logging structured events? Jump to [The Audit Trail →](#the-audit-trail)

---

Nobody sits down to build an app and thinks "I can't wait to set up logging." It is the least glamorous part of any project. No one brags about their log format at a hackathon. And yet — when an account gets compromised, when a user calls screaming that someone drained their balance — logs are the only thing standing between you and complete blindness. Every developer who has ever had to investigate an incident without them has the same story: they wished they had done it right from the start.

Think about any crime show where investigators pull surveillance footage, phone records, and transaction history to reconstruct exactly what happened. Logs are that for your application. Every login attempt, every data access, every failed request — all of it leaves a trace. Without logs, an incident investigation is guesswork. With them, you can reconstruct exactly what an attacker did, when they got in, what they touched, and how to prove it. The difference between "we think we were breached" and "here is exactly what happened and here is the evidence" is logs.

You saw in the [Dependency Security →](dependencies.md) section how a trusted package in your own supply chain can become the attack vector. The network is the same problem at a different layer. Logs are what ties it all together — the record that tells you when something in that chain was exploited, how far it got, and what it touched.

So before we get into what to log, what never to log, and how to structure it — you need to understand that logging is not one thing. Most developers treat it like it is, and that is where the problems start.

---

## The Four Types of Logs

**Operational logs** — What the infrastructure is doing. CPU usage, memory, response times, uptime, deployment events, server restarts. This is what your ops team or monitoring dashboard watches. It contains no user data and does not need to.

**Application logs** — What your code is doing, or not doing. Errors, warnings, exceptions, slow queries, failed jobs. This is what developers look at when something breaks. It should contain enough context to reproduce and fix the problem — but not passwords, tokens, or user PII.

**Audit logs** — Logs are receipts. Your application is constantly making decisions — who logged in, who did what, what they accessed, what changed, what failed. Without logs, those decisions happen and disappear. Immutable. Append-only. This is the compliance and security record. It answers the question "what happened on this account and when?" They are your receipts. And just like in real life, never show up without them.

**User-facing activity logs** — This is what the user themselves can see about their own account. Login history, active sessions, recent actions, connected devices. This builds trust and gives users the ability to spot unauthorized access themselves. Think Google's "last account activity" or the session list in a password manager.

Each of these serves a different audience, lives in a different place, and plays by different rules around retention and access. Treat them as separate systems — because they are.

---

## Log Management Tools — You Do Not Have to Build Everything Yourself

Once you understand the four types of logs, the next question is where they actually live and who manages them. You have two options: build and manage your own logging infrastructure, or use a tool that handles collection, storage, search, and alerting for you.

If you do not want to set it up yourself, these are the most common tools:

| Tool | Best For | Cost |
|---|---|---|
| **Wazuh** | Open-source SIEM, security event monitoring, compliance | Free, self-hosted |
| **Windows Event Viewer** | Windows system and application logs, built-in | Free, built-in |
| **Loki + Grafana** | Lightweight log aggregation, works well with Prometheus | Free, self-hosted |
| **Datadog** | Full observability, alerting, dashboards, easy setup | Paid, expensive at scale |
| **Papertrail** | Simple remote log aggregation, fast search | Free tier, paid above limits |
| **Splunk** | Enterprise-grade log analysis, compliance reporting | Expensive, enterprise focused |

!!! warning "The cost of convenience"

    Using a third-party log management tool means your logs — which may contain sensitive event data — are leaving your infrastructure and going to an outside vendor. That is a supply chain dependency, a data sharing arrangement, and an increased cost. Vet the vendor. Understand their data retention and access policies. Make sure their security posture matches yours.

---

## Your Data, Your Liability

Not every application has the same logging obligations. The more sensitive the data your application handles, the stricter your requirements — for what you log, how long you keep it, how you protect it, and what rights users have over it. Here are some real examples of apps people actually build — and what their data actually looks like.

**A savings calculator** that takes numbers and returns math stores nothing about you. No account, no PII, no logging obligations beyond basic operational monitoring.

**A personal trainer app** collects your name, birthday, email, weight, and workout history. Birthday and weight start crossing into health-adjacent data. If you add injury tracking or medication reminders, you are in health data territory. Log the events only — account creation, data changes, login activity — never the PII or actual workout data itself.

**A therapy or mental health app** — anything like BetterMe, Calm, or a journaling app with mood tracking — is handling some of the most sensitive data that exists. Session notes, mental health assessments, mood history. In the US, depending on how the service is structured, HIPAA may apply. In the EU, this is special category data under GDPR with the highest level of protection. Your audit log needs to track every access to this data. Your application logs must never contain it.

**A bill reader or financial aggregator** like Rocket Money reads your bank statements, spending history, and credit data — and to verify who you are, it collects significant PII: full name, address, SSN, card numbers, and more. This triggers FCRA obligations in the US. Log who accessed financial summaries and when. Never log raw account numbers, card numbers, SSNs, addresses, or any verification data in application logs. Mask everything.

**A crypto wallet app** falls under the same financial rules — and then some. Add passport scans, selfies, transaction history, and wallet addresses to the mix. KYC (Know Your Customer) and AML (Anti-Money Laundering) regulations require you to keep certain records for years. A breach here means funds can be stolen directly. Your audit log is also a regulatory requirement, not just a best practice.

!!! warning "When in doubt, log the event — not the data"

    When retaining records you need the time, date, device, and browser of when a user accessed their account or signed in — not the monthly statement they accessed. Log the action. Never the content.

---

## What to Log

Every security-relevant action in your application should leave a trace. Not the data itself — the event.

### Authentication and authorization events

Every action that touches access and identity:

```python
AUTH_EVENTS = [
    "login_success",
    "login_failure",
    "logout",
    "password_change",
    "password_reset_requested",
    "mfa_enabled",
    "mfa_disabled",
    "mfa_challenge_failed",
    "session_expired",
    "account_locked",
    "permission_granted",
    "permission_revoked",
]
```

### Data access and modification

Who touched sensitive data and what changed — never what was in it:

```python
def log_data_access(user_id, resource_type, resource_id, action, ip_address):
    log_event({
        "timestamp": datetime.utcnow().isoformat(),
        "level": "INFO",
        "event_type": "data_access",
        "user_id": user_id,
        "resource_type": resource_type,
        "resource_id": resource_id,
        "action": action,
        "ip_address": ip_address,
        "outcome": "success"
    })
```

### Security events

Anything that looks like an attack, a misconfiguration, or an anomaly:

```python
SECURITY_EVENTS = [
    "rate_limit_exceeded",
    "invalid_token_presented",
    "sql_injection_attempt_blocked",
    "csrf_validation_failed",
    "suspicious_pattern_detected",
    "file_upload_rejected",
    "dependency_vulnerability_detected",
]
```

### System and deployment events

Every change to the running state of your application:

```python
SYSTEM_EVENTS = [
    "application_startup",
    "application_shutdown",
    "config_change",
    "deployment",
    "database_migration_run",
    "backup_completed",
    "backup_failed",
    "dependency_updated",
]
```

---

## What Never to Log

Logs get breached too. They get subpoenaed. They get misconfigured and exposed. Whatever you log, assume someone else may gain access to it.

```python
# Never log any of these — not in application logs, not in audit logs, not anywhere

NEVER_LOG = [
    "passwords (plaintext or hashed)",
    "full API keys or tokens",
    "full credit card numbers",
    "CVV codes",
    "social security numbers",
    "passport numbers",
    "full session tokens",
    "OAuth access tokens",
    "private keys",
    "encryption keys",
    "full bank account numbers",
    "medical record contents",
    "therapy session notes",
    "unmasked email addresses beyond what is necessary",
    "precise geolocation beyond what is necessary",
]
```

Log that a password was changed. Not what the old or new password was. Log that a payment was processed. Not the card number. Log that a user accessed their health record. Not what was in it.

### Masking PII you do need to log

Sometimes you need just enough identifying information to be useful — to know which user, which card, which device — without storing the full value. Masking lets you keep the event meaningful without keeping the data dangerous.

```python
def mask_email(email):
    # jane.doe@example.com → j*****e@example.com
    local, domain = email.split("@")
    masked = local[0] + "*" * (len(local) - 2) + local[-1]
    return f"{masked}@{domain}"

def mask_card(card_number):
    # Only keep last 4 digits
    return "*" * 12 + card_number[-4:]

def mask_ip(ip_address):
    # Mask last octet for privacy
    parts = ip_address.split(".")
    return ".".join(parts[:3]) + ".xxx"
```

---

## Structured Logging — Format That Can Be Searched

Now that you know what to log and what to mask, the next question is how to format it — because getting the right data into your logs is only half the work. The other half is making sure it is in a format you can actually search, filter, and alert when you need it.

Every log entry should contain:

```python
import json
import logging
from datetime import datetime

def log_event(event_data):
    entry = {
        "timestamp": datetime.utcnow().isoformat() + "Z",
        "level": event_data.get("level", "INFO"),
        "event_type": event_data["event_type"],
        "user_id": event_data.get("user_id"),
        "ip_address": event_data.get("ip_address"),
        "request_id": event_data.get("request_id"),
        "outcome": event_data.get("outcome"),
        "metadata": event_data.get("metadata", {}),
    }
    entry = {k: v for k, v in entry.items() if v is not None}
    print(json.dumps(entry))
```

A login failure looks like this:

```json
{
  "timestamp": "2026-04-01T14:23:11Z",
  "level": "WARNING",
  "event_type": "login_failure",
  "user_id": "usr_8f3k2p",
  "ip_address": "203.0.113.42",
  "request_id": "req_9d2a1c",
  "outcome": "failure",
  "metadata": {
    "reason": "invalid_password",
    "attempt_count": 3
  }
}
```

No name. No email. No password. Just enough to know what happened, who it happened to by internal ID, and where it came from.

---

## Log Levels — Using Them Correctly

Not everything that goes into a log carries the same weight. Log levels are how you tell the difference between noise and a five alarm fire.

```
DEBUG     Development only. Never in production. Contains verbose internal state
          that is useful for tracing logic but exposes too much in a live system.

INFO      Normal operations. Startup, successful logins, completed jobs,
          routine system events.

WARNING   Something unexpected happened but the application kept running.
          Failed login attempts, rate limit approaches, deprecated API usage.

ERROR     Something failed. A request could not be completed, a job failed,
          an integration returned an unexpected error.

CRITICAL  The application cannot continue. Database unreachable, required
          config missing, unrecoverable state.
```

!!! warning "DEBUG in production is a security risk"

    DEBUG logs frequently contain internal state, query results, stack traces with file paths, and variable values that give an attacker a detailed map of your application. Set your log level to INFO in production. Always. A few other things to lock down:

    - **Enforce log level via environment** — set `LOG_LEVEL=INFO` in your production environment so it cannot be accidentally shipped as DEBUG
    - **Strip debug endpoints** — `/debug`, `/trace`, `/env` routes exposed by frameworks in development should never exist in production
    - **Never return stack traces to users** — log the full error server-side, return a generic message to the user

---

## Privacy, Compliance, and the Right to Be Forgotten

There is already an epidemic of stolen and leaked data — but the uncomfortable truth is that most user data is not stolen. It is sold. Companies collect it, monetize it, and move on. Beyond being low integrity and putting your users at risk, depending on what you collect and where you operate, it can also be illegal — and the lawsuits and fines are real.

GDPR in Europe and CCPA in California are just two examples of regulations that give users explicit rights over their data — including the right to have it deleted. But they are far from the only ones. We cover the full breakdown in the compliance reference below.

This creates a tension worth understanding: audit logs must be immutable, but PII must be erasable. The solution is not deletion — it is anonymization.

When a user deletes their account, the audit trail does not disappear — the identifying information does.

```python
def anonymize_user_logs(user_id):
    """
    Replace PII with anonymized placeholder.
    Keeps the event record intact for audit purposes.
    Removes the ability to identify the individual.
    """
    db.execute("""
        UPDATE audit_log
        SET
            user_id = 'deleted_user_' || id,
            ip_address = 'anonymized',
            metadata = json_patch(metadata, '{"email": "anonymized", "name": "anonymized"}')
        WHERE user_id = ?
    """, (user_id,))
    db.commit()
```

The audit event still exists. The sequence of actions is still there. The individual is no longer identifiable. You have met your erasure obligation without destroying evidence.

### Retention periods

How long you keep logs is not just an operational decision — it is a legal one.

```
Operational logs:    7–30 days. No PII. Short retention is fine.
Application logs:    30–90 days. Minimal PII. Long enough to debug recent issues.
Audit logs:          1–7 years depending on industry and jurisdiction.
                     Financial services: often 7 years minimum.
                     Healthcare: HIPAA requires 6 years.
                     General: GDPR says no longer than necessary.
User activity logs:  As long as the account exists, then anonymized on deletion.
```

---

## Compliance Reference — Know Which Rules Apply to You

You do not need to be a lawyer to build compliant software. You do need to know which regulations apply to what you are building so you are not accidentally breaking laws you did not know existed.

| Regulation | Who It Applies To | What It Covers | Key Retention Requirement | Penalty |
|---|---|---|---|---|
| **GDPR** | Any app with EU users | All personal data, right to erasure, data minimization | No longer than necessary | Up to €20M or 4% of global revenue |
| **CCPA** | Apps serving California residents with 100K+ users or $25M+ revenue | Personal data, right to deletion, opt-out of sale | 12 months minimum | Up to $7,500 per intentional violation |
| **HIPAA** | Health apps, medical records, anything handling PHI | Protected health information | 6 years | Up to $1.9M per violation category per year |
| **FCRA** | Apps accessing credit reports, financial history | Consumer financial data | 7 years for most records | Up to $1,000 per violation + damages |
| **PCI-DSS** | Any app processing card payments | Cardholder data, transaction records | 1 year online, 3 years archived | Fines + loss of ability to process cards |
| **COPPA** | Apps with users under 13 | Children's personal data | No longer than necessary | Up to $51,744 per violation |
| **SOX** | Publicly traded companies | Financial records and audit trails | 7 years | Criminal charges, up to 20 years imprisonment |

!!! warning "This is not legal advice"

    Regulations change, thresholds vary, and your specific situation may be more complex than a table can capture. If you are building something that handles health, financial, or children's data — consult a lawyer.

---

## The Audit Trail

The audit trail is what saves you or buries you. When a regulator comes knocking, when a user disputes a transaction, when a lawyer asks what happened on a specific account on a specific date — this is the record that either answers the question or doesn't.

### Immutability

Audit logs must be append-only. Insert only. No updates. No deletes. If your application user has UPDATE or DELETE permission on the audit log table, revoke it.

```sql
-- PostgreSQL — revoke modification rights on audit table
REVOKE UPDATE, DELETE ON audit_log FROM app_user;
GRANT INSERT, SELECT ON audit_log TO app_user;
```

For higher-security requirements, write audit logs to a separate system that your application cannot modify at all — a separate database, a write-once log aggregation service, or an append-only S3 bucket with object lock enabled.

### What a complete audit trail looks like

```
2026-04-01 09:14:22  user_8f3k  LOGIN          ip: 203.0.113.42   success
2026-04-01 09:15:01  user_8f3k  VIEW_RECORD    resource: financial_summary  success
2026-04-01 09:15:44  user_8f3k  UPDATE_PROFILE field: email   success
2026-04-01 09:22:11  user_8f3k  LOGOUT         success
2026-04-01 11:33:05  user_8f3k  LOGIN          ip: 198.51.100.7   success
2026-04-01 11:33:09  user_8f3k  EXPORT_DATA    format: csv        success
2026-04-01 11:33:12  user_8f3k  LOGOUT         success
```

Two logins from two different IP addresses, eleven minutes apart, with a data export in between. That pattern should trigger an alert. The audit trail made it visible.

---

## Log Storage, Rotation, and Tamper Evidence

Building the right logs is step one. Making sure they survive — and stay out of the wrong hands — is step two.

### Centralized log aggregation

Whether you are using one of the tools covered above or a simple remote syslog configuration, the goal is the same — your logs need to exist somewhere an attacker cannot reach even if they own your server.

```python
# Python — structured logging with remote handler
import logging
import logging.handlers

def setup_logging():
    logger = logging.getLogger("app")
    logger.setLevel(logging.INFO)

    local_handler = logging.StreamHandler()
    remote_handler = logging.handlers.SysLogHandler(
        address=("logs.yourloghost.com", 514)
    )

    formatter = logging.Formatter('%(message)s')
    local_handler.setFormatter(formatter)
    remote_handler.setFormatter(formatter)

    logger.addHandler(local_handler)
    logger.addHandler(remote_handler)

    return logger
```

### Log rotation

Logs that are never rotated will eventually fill your disk and take down your application just as effectively as any attack.

```bash
# /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 640 appuser appgroup
}
```

---

## Alerting on Log Events

Rotation keeps your logs manageable. Alerting is what makes them actionable — because a log nobody reads until after the incident is just an expensive paper trail.

```python
IMMEDIATE_ALERTS = [
    "login_failure",          # more than 5 in 10 minutes from same IP
    "account_locked",         # always
    "mfa_disabled",           # always — potential account takeover
    "permission_granted",     # always — privilege escalation
    "backup_failed",          # always
    "dependency_vulnerability_detected",
    "suspicious_pattern_detected",
    "database_migration_run", # in production — should never be a surprise
]
```

Connect your alerting to your notification system. The CVE Security Intelligence Monitor, an open-source security tool I built at `commit-issues/cve-security-monitor`, demonstrates this pattern — security events detected by the monitor feed directly into desktop and email notifications so nothing requires manual review to surface.

---

## Public Repos, Pull Requests, and the Audit Trail

Everything we have covered so far audits what your application does at runtime — who logged in, what they accessed, what changed. But if your repo is public and accepts pull requests, there is a layer above that which needs the same scrutiny: who is changing the code that determines what your application does, how it behaves, and what it is allowed to access in the first place.

### What the PR audit trail covers

Every merge to your main branch should be as traceable as every login to your application.

```
Who proposed the change        — GitHub username, linked to a real identity
What the change contained      — diff, files modified, code added
Whether it was reviewed        — at least one approval from a trusted reviewer
Whether it passed checks       — CI green, security scans passed, no new vulnerabilities
Whether the commit was signed  — GPG or SSH signature verifying the author
When it was merged             — timestamp, by whom
```

### Signed commits as audit evidence

As covered in the [GitHub Settings →](../hardening/settings.md) section, requiring signed commits means every commit in your history is cryptographically tied to a verified identity. A signed commit log is an audit trail. An unsigned one is a list of claims anyone could have made.

```bash
# Verify the signature on a commit
git verify-commit <commit-hash>

# See signatures in the log
git log --show-signature
```

### Not accepting contributions blindly

A public repo that merges PRs without review, without CI checks, and without signed commits is a repo that an attacker can contribute malicious code to — and has. The XZ Utils attack, covered in [Dependency Security →](dependencies.md), used exactly this vector: build trust over time, then submit a malicious change that gets merged without sufficient scrutiny.

Your branch protection rules, required reviews, and CI checks are not overhead. They are your audit controls for the codebase itself. Every PR that merges without a review is a gap in your audit trail.

---

## Ongoing Maintenance — Logging Is Not Set and Forget

Your logging system needs the same ongoing attention as your dependencies and your security posture.

### Automated scans vs scheduled reviews

**Automated scans** run on every commit and PR via CI — catching vulnerabilities, exposed secrets, and dependency issues without any manual effort. That is the CI workflow already covered above.

**Scheduled log reviews** are manual — you sit down, look at patterns, and catch what automation missed. Schedule it. Put it in your calendar.

### Dependency and security scanning in CI

```yaml
# .github/workflows/security.yml
name: Security checks

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683

      - name: Dependency audit
        run: pip-audit --requirement requirements.txt

      - name: Check for secrets in code
        run: |
          pip3 install detect-secrets
          detect-secrets scan --baseline .secrets.baseline

      - name: Run tests
        run: pytest --tb=short
```

### Tracking changes through tickets

Every security-relevant change — a dependency update, a permission change, a new data field being collected — should have a corresponding ticket. The ticket is the paper trail that connects the audit log entry ("dependency updated: requests 2.28.0 → 2.31.0") to the decision that caused it ("ticket #142: update requests to patch CVE-2023-32681").

Without tickets, your audit log tells you what happened. Tickets tell you why.

---

## When Something Goes Wrong

Logs are most valuable when you need them most — after an incident. Here is how to use them.

### Reconstructing an incident

Start from what you know — a time, an IP, a user ID, an anomalous event — and work outward:

```python
def investigate_ip(conn, ip_address, start_time, end_time):
    return conn.execute("""
        SELECT timestamp, event_type, user_id, action, outcome, metadata
        FROM audit_log
        WHERE ip_address = ?
        AND timestamp BETWEEN ? AND ?
        ORDER BY timestamp ASC
    """, (ip_address, start_time, end_time)).fetchall()

def investigate_account(conn, user_id, start_time, end_time):
    return conn.execute("""
        SELECT timestamp, event_type, ip_address, action, outcome
        FROM audit_log
        WHERE user_id = ?
        AND timestamp BETWEEN ? AND ?
        ORDER BY timestamp ASC
    """, (user_id, start_time, end_time)).fetchall()
```

### Preserving logs as evidence

If you suspect a breach, stop writing to the affected log files immediately and preserve a copy before anything else changes. Hash the files to establish that they have not been modified:

```bash
cp /var/log/myapp/audit.log /secure/evidence/audit_$(date +%Y%m%d_%H%M%S).log
sha256sum /secure/evidence/audit_*.log > /secure/evidence/checksums.txt
```

### Chain of custody

Think of every true crime case thrown out because evidence was mishandled — not because the suspect was innocent, but because the chain of custody was broken. The Golden State Killer case used DNA evidence preserved with documented chain of custody, which is exactly why it was admissible decades later. In almost every high-profile cybercrime prosecution, the defense attacks the evidence handling before the evidence itself. Document who touches your logs, when, and why — from the moment you identify the incident. That documentation is what makes the evidence hold up.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
