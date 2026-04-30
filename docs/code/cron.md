# Cron & Scheduled Tasks

You built a tool that needs to do something on its own — check for new CVEs every morning, clean up old logs every week, send a notification every time something changes. You don't want to manually run a script every day. You want it to just... happen. That's what cron is for. It's the part of your system that works while you're not looking.

Think of a cron job like setting an alarm for the most important day of your year. You don't just set the time — you make sure the volume is up, the phone is charged, and the screen is locked so nobody can sneak in and silence it, change the time, or shut it off entirely. Cron works the same way. Setting the schedule is only the first step. The rest of this section is everything else you need to make sure it actually goes off — and that nobody can tamper with it before it does.

---

## What Cron Actually Is

A cron job is a scheduled reminder to your system to run a specific task at a specific time. You set the schedule once — run this script every morning at 9am, clean up these logs every Sunday, check for updates every hour. When the time comes, your system runs the job automatically whether you're at your desk, asleep, or on vacation. You don't have to remember. You don't have to be there.

On Linux and macOS, `cron` is the built-in scheduler. On Windows, the equivalent is Task Scheduler. On GitHub, it's a workflow with a `schedule` trigger that runs on GitHub's infrastructure automatically.

The reason security matters here is simple: a job that runs without you is only as reliable and safe as the instructions you left behind. Set it up right and it runs like clockwork. Set it up carelessly and you've handed the keys to something that never sleeps, never asks questions, and never second-guesses what it's been told to do.

---

## How the Schedule Works

Cron uses a five-field time expression to define when a job runs. Think of it like filling out a form — each field is a different unit of time, and a `*` means "every":

```
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

Common examples in plain English:

```bash
# Every day at 9am
0 9 * * * /usr/bin/python3 /home/youruser/scripts/backup.py

# Every hour on the hour
0 * * * * /usr/bin/python3 /home/youruser/scripts/scan.py

# Every Monday at midnight
0 0 * * 1 /usr/bin/python3 /home/youruser/scripts/cleanup.py

# Every 30 minutes
*/30 * * * * /usr/bin/python3 /home/youruser/scripts/check.py
```

If the syntax feels like reading a different language, [crontab.guru](https://crontab.guru) is a free visual editor where you can build and test cron expressions and see exactly what they mean in plain English before you commit to them. Bookmark it.

---

## Use Absolute Paths — Always

Here's a common frustration: you write a cron job, test it manually and it works perfectly, schedule it — and it silently fails every single time. No error. No output. Nothing.

Nine times out of ten it's because of paths. Cron runs in a stripped-down environment that doesn't load your shell's settings or your PATH. It doesn't know where `python3` lives the way your terminal does. If you just write `python3`, cron looks for it in a very short list of default locations — and probably doesn't find it.

The fix is simple: always use the full path to everything.

=== "macOS / Linux / Kali"
    ```bash
    # Wrong — cron may not find python3
    python3 /home/youruser/scripts/backup.py

    # Right — full path to the binary
    /usr/bin/python3 /home/youruser/scripts/backup.py

    # Using a virtual environment — point directly to the venv's Python
    /home/youruser/project/venv/bin/python3 /home/youruser/project/scripts/backup.py

    # Not sure where a binary lives? Ask your terminal
    which python3
    which pip3
    which bash
    ```

=== "Windows (WSL2)"
    ```bash
    # Inside WSL2 — uses python3, same as Linux
    /usr/bin/python3 /home/youruser/scripts/backup.py

    # Using a virtual environment
    /home/youruser/project/venv/bin/python3 /home/youruser/project/scripts/backup.py

    # Find the full path of a binary inside WSL2
    which python3
    which pip3
    ```

=== "Windows (Native)"
    ```powershell
    # Windows uses 'python' not 'python3' in most installations
    C:\Users\youruser\AppData\Local\Programs\Python\Python311\python.exe C:\scripts\backup.py

    # Find the full path of a binary on Windows
    where python
    where pip
    ```

Apply the same rule inside your scripts — any command your script calls should use its full path.

---

## Never Put Credentials in Your Script

Imagine writing your WiFi password on a sticky note and taping it to the front door. Anyone who walks by can read it. That's what hardcoding a credential into a cron script is — except the door is your codebase, and anyone with file system access, repo access, or the ability to view your git history is walking by.

Credentials belong in a `.env` file with locked-down permissions (`chmod 600`). Your script reads them from there at runtime — they never live in the script itself. The full setup process is covered in [Credential Management](credentials.md) — follow that before wiring up any cron script that needs to authenticate.

---

## Use Lock Files to Prevent Overlap

Picture this: you schedule a job to run every 30 minutes. One day it takes 45 minutes to finish. Without a lock file, your system doesn't wait — it starts a second copy of the job while the first is still running. Now two instances are writing to the same files, hitting the same APIs, updating the same database at the same time. Data gets corrupted. Things break in ways that are hard to trace.

A lock file is the solution. When the script starts, it creates a small file. When it finishes, it deletes that file. If a second instance tries to start and finds the file already there, it exits cleanly and waits for the next scheduled run.

For bash scripts:

```bash
#!/bin/bash
LOCKFILE="/tmp/your-script.lock"

if [ -f "$LOCKFILE" ]; then
    echo "$(date): Already running. Exiting."
    exit 1
fi

touch "$LOCKFILE"
trap "rm -f $LOCKFILE" EXIT

# Your script logic below this line
```

The `trap` line is important — it makes sure the lock file gets deleted even if the script crashes halfway through.

For Python scripts:

```python
import os
import sys

LOCKFILE = "/tmp/your-script.lock"

if os.path.exists(LOCKFILE):
    print("Already running. Exiting.")
    sys.exit(1)

try:
    open(LOCKFILE, 'w').close()
    # Your script logic here
finally:
    os.remove(LOCKFILE)
```

---

## Log Everything

Running a cron job without logging is like sending someone to do an errand and never asking if they made it back. You assume it went fine until something is obviously wrong — and by then you have no record of what happened or when it started going wrong.

Log both normal output and errors to a file so you always have a record.

```bash
# Add this to your crontab entry — captures everything
0 9 * * * /usr/bin/python3 /home/youruser/scripts/backup.py >> /home/youruser/logs/backup.log 2>&1
```

The `>> /home/youruser/logs/backup.log` sends all standard output to your log file. The `2>&1` is the critical part — it redirects error output to the same place. Without it, any error your script throws disappears completely. No file. No terminal. Gone. You'll never know the job failed.

Create your log directory before the first run:

```bash
mkdir -p /home/youruser/logs
chmod 700 /home/youruser/logs
```

> 💡 **Why `700` for the directory and `600` for `.env`?**
> `chmod 600` means only you can read and write the file — no execute needed for a file you're just reading. `chmod 700` means only you can read, write, and enter the directory — directories need the execute bit set to be accessible at all. Without it, even you can't list what's inside. Use `600` for files, `700` for directories.

For structured logging inside your scripts — timestamps, log levels, formatting — see [Logging, Auditing & Accountability](logging.md). The cron-specific piece is the `2>&1` in your crontab entry. That's what determines whether errors ever reach your log file at all.

---

## Lock Down the Script File Itself

Your cron job is only as trustworthy as the script it runs. If that script lives in a folder that other users can write to, they can change what your cron job does — without ever touching the crontab entry. The schedule stays the same. The behavior changes. You won't know.

```bash
# Check who can write to your script
ls -la /home/youruser/scripts/backup.py
# The group and other columns should NOT have write (w) permission

# Fix it if they do
chmod 700 /home/youruser/scripts/backup.py
```

Keep your scripts in your home directory or a directory only you own and control.

---

## Setting Up Your Crontab

=== "macOS / Linux / Kali"
    ```bash
    # Open your crontab for editing
    crontab -e

    # View what's currently scheduled without editing
    crontab -l

    # A complete, secure crontab entry looks like this:
    0 9 * * * /usr/bin/python3 /home/youruser/project/scripts/backup.py >> /home/youruser/logs/backup.log 2>&1
    ```

=== "Windows (WSL2)"
    ```bash
    # Inside WSL2 — same crontab commands
    crontab -e
    crontab -l

    # For Windows-native scheduling, use Task Scheduler outside WSL2
    # Search: Task Scheduler → Create Basic Task
    # Point it to your script via: wsl.exe /path/to/script.sh
    ```

> 💡 **Always test before you schedule**
> Run the exact command from your crontab entry directly in your terminal first. If it works there with absolute paths, it'll work in cron. If it doesn't work in the terminal, it definitely won't work in cron — and you won't get an error message to tell you why.

---

## GitHub Actions Scheduled Workflows

A scheduled GitHub Actions workflow is cron on GitHub's infrastructure — and it has access to your repo secrets. All the same rules apply, with a few platform-specific additions.

```yaml
name: Scheduled Scan

on:
  schedule:
    - cron: '0 9 * * *'  # Daily at 9am UTC

permissions:
  contents: read          # Set this explicitly — never rely on defaults

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683  # SHA not tag
      - name: Run scan
        run: python3 scripts/scan.py
        env:
          API_KEY: ${{ secrets.API_KEY }}  # Always from secrets — never hardcoded
```

Three rules that matter most here:

- **Pin Actions to a SHA, not a tag.** Tags can be silently moved to point to different code — a SHA cannot. See [Advanced Security](../hardening/advanced-security.md) for exactly how to find and use SHAs.
- **Set permissions explicitly.** The default permissions are broader than most workflows need. Write them out and give each job only what it actually requires.
- **Use secrets for credentials.** Never hardcode an API key or token in a workflow file. Use `${{ secrets.YOUR_SECRET }}` and configure the secret in your repo settings.

---

## Cron Setup Checklist

```
□ Absolute paths used for all binaries and file references
□ Virtual environment Python path used if applicable
□ No credentials hardcoded — .env file with chmod 600
□ Lock files implemented to prevent overlapping runs
□ All output logged — stdout and stderr captured with 2>&1
□ Log directory created with appropriate permissions
□ Script files in non-writable paths — permissions verified
□ Crontab entry tested manually before scheduling
□ GitHub Actions workflows pinned to SHAs not tags
□ Scheduled workflow permissions set explicitly to minimum
□ Workflow secrets scoped to only what the job needs
```

---

## Where This Goes Next

Your cron jobs are set up securely. Once your project is running, you'll want to audit what's actually executing over time — and understand how attackers target scheduled tasks when they get access to a system.

[→ Cron & Automation (Maintenance)](../maintenance/cron.md) — How to audit your scheduled tasks and what attackers look for when they get in.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
