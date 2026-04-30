# Cron & Automation

If you've ever done a CTF, you know the move: get a foothold, run `crontab -l`, check `/etc/cron*`, and look for anything running as root or pulling from a writable path. Nine times out of ten, that's your privilege escalation. That's not a CTF trick — that's a real attack pattern. Cron jobs are one of the first things an attacker checks because they're one of the last things a developer thinks about after setup.

Here's why that matters beyond the CTF: once an attacker has planted something in your cron table or your scheduled workflow, removing the original malware isn't enough. The cron job reinstalls it. Every time. On every reboot. This is why incident response always includes auditing scheduled tasks — it's not just cleanup, it's where the backdoor lives. A rootkit that survives a reboot, a keylogger that starts on login, a reverse shell that phones home every five minutes — all of these use cron and scheduled tasks as their persistence mechanism. The malware is the payload. Cron is what keeps it alive.

---

## What Cron and Automation Actually Are

A cron job is a scheduled task — a command or script that runs automatically at a defined interval without any human input. On Linux and macOS, cron is the standard scheduler. On Windows, it's Task Scheduler. On GitHub, it's a workflow with a `schedule` trigger.

They're useful. They're also invisible. Once configured, they run in the background without prompts, without UI, without confirmation. Nobody watches them. Nobody audits them. That combination — trusted, privileged, unmonitored — is exactly what makes them a target.

---

## Why They're a Security Surface

**They run with credentials.** A cron job that sends emails, pushes to a repo, queries an API, or interacts with a database needs credentials to do it. Those credentials are somewhere — in a `.env` file, an environment variable, hardcoded in the script, or pulled from a secrets manager. How they're stored and how they're accessed is a security decision that most people make once and never revisit.

**They have access to secrets.** GitHub Actions scheduled workflows run with access to every secret you've configured in that repo. A compromised scheduled workflow doesn't just run bad code — it runs bad code with your API keys, your deployment tokens, and your cloud credentials already loaded.

**They run silently.** A compromised cron job doesn't throw an error. It doesn't send you a notification. It does what it was told to do — which is now something different than what you intended — and then it exits cleanly. If you're not logging, you won't know.

**They're rarely audited.** You set up a cron job six months ago. When did you last look at it? When did you last check whether it's still doing what you think it's doing, running as the user you intended, with the permissions you set? For most developers the answer is never.

**They can run as root.** System-level cron jobs — in `/etc/crontab`, `/etc/cron.d/`, or added by an installer — often run with elevated privileges. A script that runs as root can do anything on the system. If that script pulls from an external source, reads from a writable path, or has any injectable input, it's a full privilege escalation waiting to happen.

---

## What Attackers Actually Do With Them

**Persistence.** The primary use. An attacker who gains access to your system — through any means — plants a cron job that re-establishes their access on a schedule. You find the malware, you remove it, you think you're clean. The cron job runs at midnight and reinstalls it. This pattern repeats until someone audits the crontab.

**Privilege escalation.** A script that runs as root and reads from a path the attacker can write to is an escalation vector. The attacker modifies the script or the file it reads, waits for the next scheduled run, and their code executes as root.

**Data exfiltration on a schedule.** A cron job that quietly copies files, dumps database contents, or sends environment variables to an external endpoint — running every hour, generating no alerts, blending into normal automation traffic.

**Cryptomining.** One of the most common payloads in compromised CI/CD environments. A scheduled GitHub Actions workflow that mines cryptocurrency on GitHub's infrastructure. Your repo looks normal. Your Actions minutes are being consumed. GitHub has rate limits and detection for this — but it happens.

**Keylogging and credential harvesting.** On a compromised local machine, a cron job that starts a keylogger on login or periodically dumps shell history and sends it out. Automated, silent, persistent.

---

## Real Risks in Your Own Setup

Beyond active attackers, here's what creates vulnerability in your own cron setup before anyone else touches it:

**Hardcoded credentials in scripts.** A backup script with an API key in the file. A notification script with an email password in plain text. Anything that authenticates and has the credential written directly into the script file is a credential waiting to be found — by an attacker with file system access, by a collaborator with more access than they need, or by your own future git commit.

**Scripts that pull from external sources at runtime.** A cron job that does `curl https://example.com/script.sh | bash` at runtime is executing whatever that URL serves at the moment it runs. If that URL is compromised — through a supply chain attack, a domain expiry, or a DNS hijack — your cron job executes attacker code with whatever privileges it has. Never pull and execute at runtime.

**No logging, no failure alerts.** A cron job that fails silently is a cron job you can't monitor. If something goes wrong — or if something is wrong — you won't know until the downstream consequence surfaces.

**Overly broad permissions.** A script that needs to read one directory but runs as a user with write access to everything. Least privilege applies to scheduled tasks the same way it applies to everything else.

**Writable script paths.** If the script your cron job runs is in a directory that other users can write to, those users can modify what the cron job does. Check permissions on every script file that a scheduled task runs.

---

## Hardening Your Cron Jobs

### Use Absolute Paths for Everything

Cron runs in a minimal environment — it doesn't load your shell's PATH. If your script uses relative paths or relies on commands being in PATH, it may fail silently or — worse — find the wrong binary.

```bash
# Wrong — cron may not find python3
python3 /home/youruser/scripts/backup.py

# Right — absolute path to the binary
/usr/bin/python3 /home/youruser/scripts/backup.py

# Find the absolute path of any binary
which python3
which pip3
```

### Use Environment Variables, Not Hardcoded Credentials

Never put credentials directly in a cron script. Load them from environment variables or a secured `.env` file with restricted permissions.

=== "macOS / Linux / Kali"
    ```bash
    # Set permissions on your .env file
    chmod 600 ~/.env

    # Source it in your cron script
    source /home/youruser/project/.env
    ```

=== "Windows (WSL2)"
    ```bash
    # Same approach inside WSL2
    chmod 600 ~/.env
    source /home/youruser/project/.env
    ```

### Use Lock Files to Prevent Overlap

If a cron job takes longer than its interval, a second instance starts before the first finishes. This can cause race conditions, duplicate operations, and unpredictable behavior. Lock files prevent it.

```bash
#!/bin/bash
LOCKFILE="/tmp/your-script.lock"

if [ -f "$LOCKFILE" ]; then
    echo "Already running. Exiting."
    exit 1
fi

touch "$LOCKFILE"
trap "rm -f $LOCKFILE" EXIT

# Your script logic here
```

### Log Everything

```bash
# Redirect output to a log file in your crontab entry
0 9 * * * /usr/bin/python3 /home/youruser/scripts/backup.py >> /home/youruser/logs/backup.log 2>&1
```

Review logs regularly. A cron job that produces no output when it should, or unexpected output when it shouldn't, is a signal worth investigating.

### Audit Your Crontab

=== "macOS / Linux / Kali"
    ```bash
    # View your user crontab
    crontab -l

    # View system-wide cron jobs
    cat /etc/crontab
    ls /etc/cron.d/
    ls /etc/cron.daily/
    ls /etc/cron.weekly/
    ls /etc/cron.monthly/
    ```

=== "Windows (WSL2)"
    ```bash
    # Inside WSL2
    crontab -l

    # Windows Task Scheduler — run in PowerShell (outside WSL2)
    Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"}
    ```

For every entry: Do you know what it does? Do you know why it exists? Do you know what credentials it uses and where they're stored? If you can't answer all three, investigate before moving on.

### GitHub Actions Scheduled Workflows

A scheduled GitHub Actions workflow is a cron job with access to your secrets. Apply the same scrutiny.

```yaml
on:
  schedule:
    - cron: '0 9 * * *'  # Runs daily at 9am UTC

permissions:
  contents: read          # Explicit, minimum required
```

Pin every Action to a SHA — not a tag. A scheduled workflow that uses mutable Action tags can silently change behavior when a tag is updated. See [Advanced Security](../hardening/advanced-security.md) for pinning guidance.

Review what secrets your scheduled workflows have access to. If a workflow only needs to read your repo, it doesn't need your deployment credentials in scope.

---

## The CVE Monitor — A Real Example

Your CVE Security Intelligence Monitor uses cron to run scheduled scans, cleanups, and notifications. The hardening decisions made during that build are a direct application of everything in this section:

- Absolute Python path in the crontab entry (`/path/to/venv/bin/python3`)
- Lock files in `scheduler.py` to prevent overlapping runs
- Credentials loaded from a `.env` with `chmod 600` permissions — never hardcoded
- Logging to a dedicated log directory with retention managed by `cleanup.py`
- `self_monitor.py` watching for anomalies in the tool's own behavior

That's what a hardened automated workflow looks like in practice. Every cron job you run should meet the same standard.

---

## Cron Security Checklist

```
□ All cron scripts use absolute paths — no relative paths, no PATH reliance
□ No credentials hardcoded in scripts — environment variables or secured .env only
□ .env files set to chmod 600
□ Lock files in place to prevent overlapping runs
□ All cron output logged with timestamps
□ Crontab audited — every entry accounted for
□ System cron directories checked — /etc/cron.d/, /etc/cron.daily/, etc.
□ Script files in non-writable paths — permissions verified
□ GitHub Actions scheduled workflows pinned to SHAs
□ Scheduled workflow permissions set explicitly to minimum required
□ No scripts that pull and execute from external URLs at runtime
□ Windows Task Scheduler audited if applicable
```

---

## Where This Goes Next

Your automation is hardened. The last two sections cover the ongoing intelligence that keeps everything else current — monitoring your dependencies after they're installed, and keeping your notification system from becoming a liability.

[→ Dependency Intelligence](dependencies.md) — Ongoing monitoring of your dependencies after installation, not just at setup.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
