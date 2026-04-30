# Backup & Recovery

You're ready to ship. You run your final checks — something fails hard. You need to roll back to the last known good state, fast. You head to GitHub, find the commit, and realize the history isn't what you thought it was. A force push from last week wiped it. You had one copy. It was on GitHub. That was your backup. Except it wasn't.

It happens faster and more often than you think. Here's how to make sure it doesn't happen to you.

---

## What GitHub Actually Protects You From

GitHub has redundancy, uptime guarantees, and infrastructure-level backups. What that means is that GitHub's servers are protected. Your data on those servers is a different story.

GitHub does not protect you from:

- A `git push --force` that overwrites history
- A deleted repository — GitHub gives you a short recovery window, but it is not guaranteed and it is not permanent
- A compromised account that deletes or corrupts your repos
- Your own mistakes — bad merges, accidental branch deletions, reset commands that went further than intended
- A GitHub outage at the exact moment you need to deploy or rollback

GitHub is a host and a collaboration platform. It is not a backup service. Treating it as one is one of the most common and most painful mistakes developers make.

---

## What Can Actually Go Wrong

These are not hypothetical. Each of these has happened to real developers on real projects:

**Force push that wiped history.** Someone on the team — or you — ran `git push --force` to a branch without thinking. The remote history was rewritten. If nobody had a local clone with the old history, it's gone.

**Deleted repository.** A repo deleted by accident, by a compromised account, or by a departing team member with admin access. GitHub's recovery window is short and not guaranteed. After that window closes, the data is gone.

**Compromised account.** An attacker who gains access to your GitHub account can delete repos, force push to branches, rotate your keys, and remove collaborators — all before you know what's happening. Your code is only as safe as your account.

**GitHub outage.** GitHub has had significant outages. If your only copy of your code is on GitHub and GitHub is down, you cannot work, deploy, or recover. Having a local copy is not optional.

**Corrupted local environment.** Your machine dies. Your drive fails. Your laptop gets stolen. If your only local copy was the working directory you never pushed, it's gone.

---

## What a Real Backup Strategy Looks Like

A real backup strategy has three layers: local, remote, and automated.

### Layer 1 — Local Bare Clone

A bare clone is a complete copy of your repository — all history, all branches, all tags — without a working directory. It's the most complete backup you can take of a Git repo.

=== "macOS / Linux / Kali"
    ```bash
    # Create a bare clone backup
    git clone --bare git@github-commitissues:commit-issues/your-repo.git ~/backups/your-repo-backup.git

    # To update an existing bare clone
    cd ~/backups/your-repo-backup.git
    git fetch --all
    ```

=== "Windows (WSL2)"
    ```bash
    # Same commands inside WSL2
    git clone --bare git@github-commitissues:commit-issues/your-repo.git ~/backups/your-repo-backup.git

    cd ~/backups/your-repo-backup.git
    git fetch --all
    ```

Store this somewhere that isn't your main machine — an external drive, a NAS, a secondary device. A backup on the same machine you're recovering from is not a backup.

### Layer 2 — Mirror to a Second Remote

Mirror your repo to a second hosting provider so that if GitHub goes down or your account is compromised, you have a complete, up-to-date copy elsewhere. GitLab offers free private repos and is the most common choice.

```bash
# Add a second remote
git remote add gitlab git@gitlab.com:yourusername/your-repo.git

# Push to both remotes
git push origin main
git push gitlab main

# Or push to all remotes at once — add this to your git config
git remote set-url --add --push origin git@gitlab.com:yourusername/your-repo.git
```

After the second line above, a single `git push origin main` will push to both GitHub and GitLab simultaneously.

### Layer 3 — Back Up What Git Doesn't Track

Your git history backs up your code. It does not back up everything your project needs to run.

**What to back up separately:**

- `.env` files — your local environment variables, API keys, and configuration. These are gitignored for good reason but need to exist somewhere safe. Use a password manager or encrypted vault — not a plain text file, not an email to yourself.
- Database files — if your project uses a local database (SQLite, encrypted DB), back it up separately and regularly. Git does not track binary files well and you should not be committing your database anyway.
- Secrets reference document — a secure, encrypted record of what credentials exist, what they're for, and where they're stored. Not the credentials themselves — the map.
- SSL certificates and signing keys — if you manage your own certificates or GPG/SSH signing keys, back them up to an encrypted location.

> ⚠️ **Never back up raw credentials to a cloud service without encryption**
> A `.env` file sitting in Google Drive or Dropbox unencrypted is an exposure waiting to happen. Use an encrypted vault — Bitwarden, 1Password, or an encrypted archive — for anything credential-adjacent.

---

## Automating Your Backups

Manual backups don't get done. Automate them.

=== "macOS"
    ```bash
    # Create a backup script
    cat > ~/scripts/backup-repos.sh << 'EOF'
    #!/bin/bash
    REPOS=("your-repo" "another-repo")
    BACKUP_DIR=~/backups

    for REPO in "${REPOS[@]}"; do
        if [ -d "$BACKUP_DIR/$REPO.git" ]; then
            cd "$BACKUP_DIR/$REPO.git"
            git fetch --all
        else
            git clone --bare git@github-commitissues:commit-issues/$REPO.git "$BACKUP_DIR/$REPO.git"
        fi
    done
    EOF

    chmod +x ~/scripts/backup-repos.sh

    # Schedule with cron — runs daily at 9am
    crontab -e
    # Add: 0 9 * * * /Users/youruser/scripts/backup-repos.sh
    ```

=== "Linux / Kali"
    ```bash
    # Same script, schedule with cron
    chmod +x ~/scripts/backup-repos.sh
    crontab -e
    # Add: 0 9 * * * /home/youruser/scripts/backup-repos.sh
    ```

=== "Windows (WSL2)"
    ```bash
    # Run the script inside WSL2 and schedule with WSL2 cron
    chmod +x ~/scripts/backup-repos.sh
    crontab -e
    # Add: 0 9 * * * /home/youruser/scripts/backup-repos.sh
    ```

See [Cron & Automation](cron.md) for hardening your scheduled jobs so the backup script itself isn't a security risk.

---

## Recovery — What To Actually Do

### Restore From a Bare Clone

If your GitHub repo is gone or corrupted and you have a bare clone:

```bash
# Push your bare clone back to a new GitHub repo
cd ~/backups/your-repo-backup.git
git push --mirror git@github-commitissues:commit-issues/your-repo.git
```

This restores all branches, all tags, and all history.

### Recover a Deleted GitHub Repository

GitHub allows repo recovery within a short window after deletion — typically 90 days for repos deleted by the owner, but this is not guaranteed and varies.

```
GitHub → Settings → Repositories → Deleted repositories
```

If the repo appears there, you can restore it. If it doesn't — or if the window has passed — your only option is a local or bare clone backup.

### Recover From a Bad Force Push

If a force push wiped history and someone on your team has a local clone with the old commits:

```bash
# On the machine with the old history
git push origin main --force

# This overwrites the bad state with the good state
# Coordinate with your team — this will affect their local copies
```

If nobody has the old history locally, the commits are gone. This is why bare clone backups matter.

---

## Backup Frequency — Two Tiers

> 💡 **These are general guidelines based on common industry practices and standards.** If you ship actively or run a project people depend on, the Paranoid & Preventative column is where you want to be.

| What | Industry Standard | Paranoid & Preventative |
|------|-------------------|------------------------|
| Local bare clone update | Weekly | Daily |
| Mirror push to second remote | Every push | Every push — automate it |
| `.env` and secrets vault backup | Monthly | Weekly |
| Database backup | Weekly | Daily or on every significant change |
| Full backup verification (restore test) | Quarterly | Monthly |

---

## Backup Checklist

```
□ Bare clone exists for every active repo
□ Bare clones stored on a separate device or drive
□ Mirror remote configured on a second hosting provider
□ Backup script created and scheduled via cron
□ .env and credentials backed up to encrypted vault
□ Database backed up separately from code
□ Deleted repo recovery window known and checked
□ Recovery process tested — not just assumed to work
```

> ⚠️ **A backup you've never tested is not a backup**
> Run a recovery drill. Restore from your bare clone to a test repo. Confirm the history, the branches, and the tags are all there. Do this before you need it — not during the incident.

---

## Where This Goes Next

Your code is backed up. Your account is the next thing that needs protecting — because everything else flows through it.

[→ Email Security](email.md) — Why your email account is the master key to your entire GitHub presence and how to lock it down.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
