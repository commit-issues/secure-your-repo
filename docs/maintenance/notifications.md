# Notifications

Your notification system is supposed to be your early warning. But if it's misconfigured, it's quietly leaking your credentials, burying you in noise until real alerts get ignored, misfiring until you stop trusting it, or failing silently while you assume you're protected. Any of the four is dangerous.

Many developers set up notifications once, confirm the first alert comes through, and never look at the configuration again. That's how you end up with a webhook URL sitting in a committed config file, an SMTP password in plain text in a script, and a Dependabot alert queue that's been firing for six weeks to a Slack channel nobody checks anymore.

This section is about building a notification system that actually works — one that delivers the right alerts to the right place, protects the credentials it runs on, and stays tuned tightly enough that you don't train yourself to ignore it.

---

## Why Notifications Are a Security Risk

The part most people miss: **a notification system is a credential pipeline.** Every alert your tool sends has to authenticate to something to get there. That authentication is a credential — and credentials can be exposed.

A Slack webhook URL is not just a URL. It is a credential. Anyone who has it can send messages to your Slack channel — spoofed alerts, fake security warnings, social engineering bait — with no authentication required beyond possessing the URL. If that URL ends up in a committed config file, a screenshot, a log output, or a shared doc, it's compromised. You may not know until someone uses it.

The same applies to every other notification channel:

- **SMTP credentials** for email alerts — username and password that authenticate to your email provider. Exposed the same way any other hardcoded credential is.
- **API tokens** for services like Mailgun, Twilio, PagerDuty — scoped tokens that can send messages, sometimes read account data, sometimes do more depending on how broadly they were issued.
- **Discord webhook URLs** — same as Slack. Possession of the URL is the only authentication.
- **Email addresses in notification configs** — the destination address you configure to receive alerts is often stored in plain text in your config or `.env` file. If that config gets committed or exposed, an attacker now knows exactly which email address your security alerts go to. That's enough to craft a phishing email that looks identical to your real alerts — same format, same subject line, same urgency — sent to you directly. You open it because it looks like every other alert you've been getting.

None of these are exotic attack vectors. They're all just credentials sitting in places where credentials shouldn't be.

---

## What Can Actually Go Wrong

### Webhook URL Committed to a Public Repo

You set up a Slack or Discord notification. You put the webhook URL in a config file. You commit the config file. Your repo is public. The URL is now in your git history — permanently, even if you delete it in the next commit.

Automated scanners find exposed webhook URLs within seconds of a push. The same tools that hunt for API keys hunt for these. Once found, the URL gets used — to spam your channel, to send fake security alerts designed to look like yours, or to probe what other information leaks through your notification pipeline.

**The fix:** Webhook URLs belong in environment variables, not config files. Treat them exactly like you treat an API key. See [Credential Management](../code/credentials.md).

```bash
# Wrong — URL in config file that gets committed
SLACK_WEBHOOK = "https://hooks.slack.com/services/T00/B00/XXXX"

# Right — loaded from environment
SLACK_WEBHOOK = os.environ.get("SLACK_WEBHOOK_URL")
```

### SMTP Credentials in Plain Text

Email notification systems need to authenticate to send. That means a username and password — or an API key — somewhere in your code or config. If it's hardcoded, it's exposed. If it's in a `.env` file without `chmod 600`, it's readable by anyone with file system access. If it's in a log file because your script accidentally printed it, it's in your logs.

Something worth considering when choosing an email notification provider: services like SendGrid, Mailgun, and others often require personal account information during signup — real name, address, phone number — that becomes tied to the credential itself. That's a privacy liability. If the credential gets exposed, so does the personal information attached to the account. Look for providers that offer clean API key scoping with minimal personal data requirements, and read their privacy policy before you wire one into your tool.

**The fix:** API keys and SMTP credentials in `.env` with `chmod 600`. Never in the script. Never in a config that gets committed. Rotate them on any suspected exposure. And choose your provider deliberately — the service you use matters, not just how you store the credentials.

### Silent Delivery Failure

This is the most dangerous failure mode because it looks like everything is fine. Your notification system is configured. You tested it once and it worked. But somewhere between then and now — a webhook URL was rotated, a token expired, an email provider blocked your sending domain, a network rule changed — and the alerts stopped arriving.

You don't know. You think you're protected. You aren't.

```bash
# Test your notification delivery explicitly — don't assume
# For the CVE monitor — run a test alert manually
python3 src/notifier.py --test

# For a webhook — send a test payload
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Notification test — if you see this, delivery is working"}' \
  $SLACK_WEBHOOK_URL
```

Build delivery verification into your maintenance schedule. If you can't confirm an alert arrived, you can't trust your notification system.

### Alert Fatigue — When the Warning Becomes Noise

Alert fatigue is a security risk disguised as an inconvenience. When your notification system fires too many alerts — false positives, low-severity signals, duplicate notifications, alerts for things that don't require action — you start filtering them out mentally. You stop reading them carefully. You dismiss them faster. And then the one alert that actually matters arrives, and it gets the same treatment as the forty that didn't.

This is not a hypothetical. Security teams at major organizations have missed critical breach indicators because they were buried in alert noise. The same pattern plays out at the individual developer level.

**Signs your alerts have become noise:**
- You see a notification from your security tool and your first instinct is to dismiss it
- You have a backlog of unread alerts you plan to "get to later"
- You've disabled a notification channel because it was "too noisy"
- You can't remember the last time an alert prompted you to actually do something

**The fix:** Tune ruthlessly. Every alert category should have a reason for existing and an action associated with it. If an alert fires and the correct response is "ignore it," that alert shouldn't fire. See the severity and filtering configuration in your tool's notification settings.

### Misfires That Train You to Ignore Real Alerts

A misfire is a false positive — an alert that fires when nothing is actually wrong. One or two are acceptable. A pattern of misfires is a calibration problem that becomes a trust problem. Once you've been burned by enough false positives, you start treating all alerts with skepticism — including the ones that are real.

The CVE Security Intelligence Monitor addresses this directly with `self_monitor.py` — a false-positive suppression layer that filters signal from noise before an alert ever fires. The goal is that when the monitor alerts you, it means something. Every alert that fires should be worth reading.

---

## Secure Notification Setup

### Store Every Credential Like It's an API Key

Webhook URLs, SMTP passwords, API tokens, email addresses used as notification destinations — all of it goes in `.env` with `chmod 600`. None of it gets committed. All of it gets rotated if there's any suspicion of exposure.

```bash
# Your .env for notification credentials
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...
RESEND_API_KEY=re_...
NOTIFICATION_EMAIL=alerts@yourdomain.com
SMTP_PASSWORD=your-smtp-password

# Lock it down
chmod 600 .env
```

Add every notification credential name to your `.gitignore` check and your pre-commit secret scanning. See [Credential Management](../code/credentials.md) and [Advanced Security](../hardening/advanced-security.md).

### Know Who Gets Notified and Why

Every notification destination is a decision. Who is receiving this alert? Do they need it? What are they supposed to do with it? A notification that goes to someone who can't act on it is just noise for them and a false sense of coverage for you.

For personal projects: one primary channel you actually monitor, one backup. Not five channels you added because they were easy to set up.

For team projects: route by severity. Critical alerts go to the person who can respond immediately. Informational alerts go to a shared channel. Low-severity summaries go in a weekly digest, not a real-time ping.

### Verify Before You Trust

After any configuration change — new webhook, new API key, new email credential, new notification channel — send a test alert and confirm it arrived. Don't assume. Confirm.

=== "macOS / Linux / Kali"
    ```bash
    # Test a webhook directly
    curl -X POST -H 'Content-type: application/json' \
      --data '{"text":"Test alert — confirm delivery"}' \
      "$SLACK_WEBHOOK_URL"

    # Test email delivery
    python3 -c "
    import smtplib
    from email.mime.text import MIMEText
    msg = MIMEText('Test notification delivery')
    msg['Subject'] = 'Notification Test'
    msg['From'] = 'your@email.com'
    msg['To'] = 'destination@email.com'
    with smtplib.SMTP_SSL('smtp.yourprovider.com', 465) as s:
        s.login('your@email.com', 'your-password')
        s.send_message(msg)
    print('Delivered')
    "
    ```

=== "Windows (WSL2)"
    ```bash
    # Same commands inside WSL2
    curl -X POST -H 'Content-type: application/json' \
      --data '{"text":"Test alert — confirm delivery"}' \
      "$SLACK_WEBHOOK_URL"
    ```

=== "Windows (Native)"
    ```powershell
    # Test a webhook from PowerShell
    Invoke-RestMethod -Uri $env:SLACK_WEBHOOK_URL `
      -Method Post `
      -ContentType 'application/json' `
      -Body '{"text":"Test alert — confirm delivery"}'
    ```

---

## The CVE Monitor — A Real Example

The CVE Security Intelligence Monitor (`commit-issues/cve-security-monitor`) implements a dual-track notification system — desktop and email — designed around the lessons in this section.

**Why dual-track:** Desktop notifications are immediate but ephemeral. If you miss it, it's gone. Email creates a record — a searchable, auditable history of every alert that fired and when. For a security tool, that audit trail matters. You want to be able to look back and confirm when you were notified about something.

**On choosing an email provider:** The CVE monitor removed email notification support via third-party providers specifically because of the privacy concerns outlined above — personal account information tied to credentials, data retention policies that didn't match the tool's privacy standards. The lesson: don't wire a notification service into a security tool without vetting how that service handles your data and your credentials. If the provider requires more personal information than the use case justifies, that's a red flag.

**How alerts are tuned:** `self_monitor.py` runs false-positive suppression before any alert fires. The notification type values (`Desktop`, `Email`, `Both`) are set explicitly in the database — not inferred, not defaulted, not guessed. Every alert that reaches you was evaluated before it was sent.

**Credentials:** Notification API keys live in `.env` with `chmod 600`. Never hardcoded, never committed, rotated on any suspected exposure.

---

## Notifications Checklist

```
□ Webhook URLs stored in .env — not hardcoded in config files
□ SMTP credentials and API tokens in .env with chmod 600
□ .env added to .gitignore — verified not committed
□ Secret scanning enabled — catches credentials before they hit the repo
□ Delivery verified after every configuration change
□ Alert channels reviewed — only channels you actively monitor
□ Notification destinations documented — who gets what and why
□ Alert severity tuned — no false positives training you to ignore alerts
□ Backup notification channel configured and tested
□ Credential rotation schedule set for notification API keys
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
