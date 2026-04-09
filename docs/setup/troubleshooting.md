# Troubleshooting & Recovery

> Something went wrong. You are not the first. Here is how to fix it.

---

!!! abstract "TL;DR"
    - Lost your SSH passphrase — you cannot recover it. Generate a new key, revoke the old one.
    - Exposed a secret — revoke it at the source first, rewrite history second.
    - Lost your .env file — reconstruct from .env.example, generate new credentials for everything.
    - Locked out of GitHub — use backup codes first, then contact GitHub support.
    - Wrong SSH key on GitHub — revoke it, add the correct one, test before assuming it works.
    - pre-commit hook blocking you — read what it found before bypassing anything.
    - Repo accidentally made public — act immediately, assume everything was seen.

    Use `Ctrl+F` or `Cmd+F` to jump to your specific issue.

---

This section exists because security is not just about doing things right the first time. It is about knowing what to do when things go wrong — because they will. Keys get lost. Secrets get committed. Accounts get locked. Repos get misconfigured. Every developer who has been building for any length of time has at least one story that starts with "I accidentally..." 

The difference between a recoverable incident and a disaster is almost always speed and knowing the right steps. This section covers the most common things that go wrong and exactly how to fix them — written by someone who has been there.

---

## Lost or Forgot Your SSH Passphrase

This is one of the most common SSH problems and it has one answer: ***you cannot recover a lost SSH passphrase***. It is not stored anywhere retrievable. There is no "forgot passphrase" option. The passphrase is used to encrypt your private key file on disk — without it, the file is unreadable.

What you can do is generate a new key and replace the old one.

**Step 1 — Generate a new SSH key:**

Follow the full process in [SSH Keys →](ssh.md) from the beginning. Use a new, strong passphrase and store it in your password manager this time.

**Step 2 — Add the new public key to GitHub:**

```
GitHub → Settings → SSH and GPG keys → New SSH key
Add the new .pub file contents as an Authentication Key
Add it again as a Signing Key
```

**Step 3 — Remove the old key from GitHub:**

```
GitHub → Settings → SSH and GPG keys
Find the old key → Delete
```

**Step 4 — Update your SSH config if you have one:**

Open `~/.ssh/config` and update the `IdentityFile` line for any affected hosts to point to the new key file.

**Step 5 — Test the new connection:**

```
ssh -T git@github.com
```

You should see your username confirmed. Only then should you consider the transition complete.

!!! warning "Do not skip revoking the old key"
    Even if you can no longer use your old key, it is still trusted by GitHub until you delete it. If the old key file exists somewhere on your machine or in a backup, anyone who finds it and knows it has no passphrase — or figures out the passphrase — can still authenticate as you. Delete it from GitHub immediately.

---

## Wrong SSH Key Added to GitHub

You added the wrong file to GitHub — maybe the private key instead of the public key, or the wrong account's key, or a key from a different machine than intended.

**How to tell if you added the wrong thing:**

If you accidentally added a private key, the contents you pasted would have started with `-----BEGIN OPENSSH PRIVATE KEY-----`. If you see that in your GitHub SSH keys page under the key content — that is a private key and it needs to be handled carefully.

**If you added a private key to GitHub:**

```
Step 1 — Delete it from GitHub immediately:
GitHub → Settings → SSH and GPG keys → Delete

Step 2 — That key is now compromised.
Generate a brand new key pair — do not reuse the old one.
The old private key must be considered burned.

Step 3 — Add the new PUBLIC key (.pub file) to GitHub.

Step 4 — Delete the old key files from your machine:
rm ~/.ssh/id_ed25519_old
rm ~/.ssh/id_ed25519_old.pub
```

**If you added a key from the wrong machine or account:**

```
Step 1 — Delete the incorrect key from GitHub
Step 2 — Add the correct public key
Step 3 — Test the connection: ssh -T git@github.com
```

!!! tip "How to verify you have the right file before adding it"
    Before you paste anything into GitHub, run this command and check the output:
    ```
    cat ~/.ssh/id_ed25519.pub
    ```
    The output should be a single line starting with `ssh-ed25519` and ending with your comment. If it is multiple lines, starts with `-----BEGIN`, or looks like random binary characters — stop. You have the wrong file.

---

## Lost Your .env File

Your `.env` file is not in git — that is by design. Which means if you delete it, lose it in a system wipe, or set up on a new machine, it is gone. There is no recovery from git history.

This is actually fine — because your `.env.example` file tells you exactly what needs to be in it. You just need new credential values.

**Step 1 — Reconstruct the structure from .env.example:**

```
cp .env.example .env
```

This gives you a new .env file with all the right variable names and placeholder values.

**Step 2 — Generate new credentials for every service:**

Do not try to remember the old values. Do not reuse them from memory. Go to each service — your API providers, database, email service, whatever your project uses — and generate fresh credentials. Treat the old ones as expired.

**Step 3 — Fill in the new .env with the fresh values.**

**Step 4 — Test your application to confirm everything connects correctly.**

!!! warning "If you lost your database encryption key"
    This is a different and more serious situation. If your database was encrypted with a key stored in .env and you lost the .env file without a backup of the key — the database may be unrecoverable. This is why the guidance throughout this guide emphasizes storing your encryption key somewhere safe and offline, separate from the .env file itself. If you are in this situation, check any offline backups, password manager entries, or encrypted notes where you might have saved it. If it is truly gone, a new database will need to be initialized from scratch.

---

## API Key or Secret Exposed in a Commit

This is the incident everyone dreads and more people have experienced than will admit. You committed a real API key, password, or token to a repository — possibly public.

***Speed is everything here. Every minute the key is live and exposed is a minute it can be harvested.***

**Step 1 — Revoke the exposed credential immediately:**

Go to whatever service issued the key — NVD, OpenAI, AWS, SendGrid, Stripe, wherever — and invalidate it right now. Before you do anything else. Before you fix the code. Before you tell anyone. Revoke it.

**Step 2 — Generate a new credential from the same service.**

**Step 3 — Update your .env with the new value.**

**Step 4 — Rewrite git history to remove the old value:**

Install git filter-repo if you do not have it:

```
macOS / Linux:
pip install git-filter-repo --break-system-packages

Windows:
pip install git-filter-repo
```

Create a file called `expressions.txt` with the exposed value:

```
your-exposed-api-key==>REDACTED
```

Then run:

```
git filter-repo --replace-text expressions.txt
```

This replaces every occurrence of the exposed key throughout your entire git history with the word REDACTED.

Alternatively, if the secret was in a specific file you want to remove entirely from history:

```
git filter-repo --path path/to/the/file --invert-paths
```

**Step 5 — Force push the clean history:**

```
git push origin main --force
```

**Step 6 — Request a cache purge from GitHub:**

Even after rewriting history, GitHub may cache old commit views. Contact GitHub support and request a cache purge for the affected repository:

```
https://support.github.com
```

**Step 7 — Audit for unauthorized use:**

Go to the service where the key was issued and check usage logs for any activity you did not perform. If you find unauthorized usage, follow that service's incident response process.

!!! warning "The key is burned even after you rewrite history"
    Rewriting history removes the secret from your repository going forward. It does not guarantee that no one copied it before you acted. Automated scanners index public repositories continuously — some within minutes of a push. The new credential you generated in Step 2 is the one that matters now. The old one is burned regardless of whether anyone used it.

---

## Locked Out of Your GitHub Account

You cannot log in — lost your 2FA device, lost your backup codes, or both.

**If you have backup codes:**

```
GitHub login page → "Use a recovery code instead"
Enter one of your backup codes
```

Once logged in, immediately:
- Generate new backup codes and store them safely
- Re-enroll your 2FA with a new authenticator app
- Review your security log for any activity during the lockout period

**If you have no backup codes and no 2FA device:**

GitHub's account recovery process requires you to verify your identity. This takes time — sometimes days. Start here:

```
https://github.com/contact
Select: "I can't sign in"
Follow the identity verification process
```

GitHub will ask you to verify ownership through things like SSH key verification, email confirmation, or other account signals. The more security you had set up, the more options you have for verification.

!!! warning "This is why backup codes matter"
    Account recovery without backup codes is slow, uncertain, and stressful. If you have not saved your backup codes yet — stop reading this and do it now. [Securing Your GitHub Account → Save Your Backup Codes](account.md#save-your-backup-codes-right-now)

**Prevention going forward:**

Once you recover access, store backup codes in at least two places — your password manager and a printed copy in a secure physical location. This is the one situation where redundancy is worth the inconvenience.

---

## Committed to the Wrong GitHub Account

You pushed commits to a repo under the wrong GitHub account — maybe your work account instead of personal, or vice versa.

**Why this happens:**

Usually a mismatch between your SSH config alias and your git remote URL, or a global git config pointing to the wrong identity. We covered setting this up correctly in [SSH Keys → Configuring for Multiple Accounts](ssh.md#configuring-sshconfig-for-multiple-accounts).

**How to check which account made a commit:**

```
git log --format="%H %ae %an" -10
```

This shows the last 10 commits with the email and name attached to each. If the wrong identity is showing — the commits are attributed to the wrong account.

**Fix the remote URL to use the right SSH alias:**

```
git remote set-url origin git@github-correctalias:yourusername/your-repo.git
```

**Fix the identity for future commits in this repo:**

```
git config user.email "correctnoreply@users.noreply.github.com"
git config user.name "correctusername"
```

**If you need to rewrite the author on past commits:**

```
git filter-repo --email-callback '
    return email if email != b"wrong@email.com" else b"correct@noreply.github.com"
'
```

Then force push:

```
git push origin main --force
```

---

## pre-commit Hook Blocking a Commit

The hook found something that looks like a credential and refused the commit. This is the hook doing exactly what it is supposed to do.

**Before you do anything else — read what it found:**

The output will tell you which file and which line triggered the detection. Go look at that line. Ask yourself honestly: is this a real secret or a false positive?

**If it is a real secret:**

Remove it from the file. Move it to your .env. Then try the commit again. The hook just saved you from an incident.

**If it is a false positive — a placeholder, an example value, or something intentional:**

Update the baseline to acknowledge it:

```
detect-secrets scan --baseline .secrets.baseline
```

Review the updated baseline to confirm it only contains things you have intentionally acknowledged. Then commit the updated baseline and try your commit again.

**If you are absolutely certain it is safe and need to bypass the hook temporarily:**

```
git commit --no-verify -S -m "your commit message"
```

!!! warning "Use --no-verify sparingly and intentionally"
    Every time you bypass the hook you are making a conscious choice to skip the safety check. That is sometimes necessary — but it should never be a habit. If you find yourself using `--no-verify` regularly, something in your workflow needs to be fixed, not bypassed. Figure out what is causing the false positives and update the baseline properly.

---

## SSH Connection Failing After Working Fine

Your SSH connection to GitHub was working and now it is not. This is usually one of a few things.

**Run the diagnostic first:**

```
ssh -vT git@github.com
```

The `-v` flag gives verbose output showing exactly where the connection process is failing. Read through it — the failure point is usually obvious once you see it.

**Common causes and fixes:**

```
"Permission denied (publickey)"
→ Your key is not loaded in the agent.
  Run: ssh-add -l
  If empty: ssh-add ~/.ssh/id_ed25519

"Could not open a connection to your authentication agent"
→ The SSH agent is not running.
  macOS / Linux: eval "$(ssh-agent -s)"
  Windows: Start-Service ssh-agent

"WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"
→ GitHub's host key changed (this does happen occasionally)
  or something suspicious is happening on your network.
  Verify GitHub's current fingerprints at:
  https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints
  If they match — update your known_hosts:
  ssh-keygen -R github.com
  Then reconnect and accept the new fingerprint.

"ssh: connect to host github.com port 22: Connection timed out"
→ Port 22 is being blocked, likely by a firewall or network.
  Try connecting over port 443:
  ssh -T -p 443 git@ssh.github.com
  If that works, update your ~/.ssh/config to use port 443 for GitHub.
```

---

## Repo Accidentally Made Public

You changed a private repo to public — or created it as public when it should have been private. Act immediately.

***Assume everything in that repo's history was seen the moment it went public. Automated scanners do not wait.***

**Step 1 — Make it private again immediately:**

```
GitHub → Settings → General → Danger Zone → Change visibility → Make private
```

**Step 2 — Audit the entire commit history for secrets:**

```
git log --all --full-history -- "**/.env" "**/*.key" "**/*.pem"
```

Also run detect-secrets against your full history:

```
detect-secrets scan --all-files
```

**Step 3 — Revoke any credentials that were in the repo at any point:**

Even if you think they were in a file that was deleted before the repo went public — git history preserves deleted files. Assume any credential that ever touched this repo is compromised.

**Step 4 — Rewrite history to remove any sensitive content:**

Follow the steps in [API Key or Secret Exposed](#api-key-or-secret-exposed-in-a-commit) above.

**Step 5 — Decide whether to make it public again:**

Only make it public again after you are certain the history is clean. When in doubt — keep it private or delete the repo and start fresh with a clean history.

---

## How to Audit Your Repo for Sensitive Data Right Now

Whether you are inheriting a repo, cleaning up an old project, or just want to verify your current state — here is how to check what is actually in your repository.

**Check what files git is tracking that should not be tracked:**

```
git ls-files | grep -E "\.(env|key|pem|p12|pfx|db|sqlite)$"
```

If anything shows up here that should not be tracked — it needs to be removed from git history, not just deleted.

**Check for secrets in your current files:**

```
detect-secrets scan --all-files
```

Review every finding. Anything that is a real credential needs to be revoked and removed.

**Check your git history for sensitive strings:**

```
git log --all -p | grep -i "api_key\|password\|secret\|token\|private_key" | head -50
```

This searches through every commit's diff for common credential-related terms. It is not exhaustive but it catches the most common patterns.

**Check your commit author history for identity leakage:**

```
git log --format="%ae" | sort | uniq
```

This lists every email address that has ever made a commit to this repo. If you see a personal email address that should not be public — follow the steps in [Committed to the Wrong Account](#committed-to-the-wrong-github-account) to rewrite it.

**Check your remote URL to confirm you are pushing to the right place:**

```
git remote -v
```

Verify the URL matches the repo and account you intend. If you have a custom SSH alias, confirm it resolves to the right account.

!!! tip "Run this audit on every repo you inherit or fork"
    Before you start working in a repo that someone else created or that was transferred to you — run through these checks. You are not just checking for your mistakes. You are checking for whoever came before you. Finding a problem before you push anything new is infinitely easier than untangling it afterward.

---

## Quick Reference — Emergency Contacts and Resources

```
GitHub Support:
https://support.github.com

GitHub Security Log (your account activity):
GitHub → Settings → Security log

GitHub SSH Key Fingerprints (verify authenticity):
https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints

Report a security vulnerability in a GitHub repo:
GitHub → repo → Security → Report a vulnerability

Have I Been Pwned (check if your email is in a breach):
https://haveibeenpwned.com
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
