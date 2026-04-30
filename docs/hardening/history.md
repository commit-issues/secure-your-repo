# Git History Auditing

> Your history does not forget. The secrets from your past are still there — do you know what's hiding in yours?

---

!!! abstract "TL;DR"
    - git history is permanent — deleting a file does not remove it from history.
    - Every email, username, file path, and secret ever committed is still there.
    - Run the full audit on every repo before calling it clean.
    - git filter-repo is the correct tool for rewriting history — not git filter-branch.
    - After any rewrite — force push, request a GitHub cache purge, notify forks.
    - Prevention is a git config check before every first commit on every machine.
    - If something went public — act immediately, assume it was seen.

    Already know the basics? Jump to [The Full Audit Checklist](#the-full-audit-checklist)

---

Git history is a permanent record of everything — including every mistake. Every email address, every username, every accidentally committed secret, every path that revealed your machine name, every commit made from the wrong account. Most people never look at their history from a security perspective. This section is the full audit — what to look for, how to find it, how to fix it, and why "I deleted the file" is never enough.

This is not a theoretical exercise. Even developers who care deeply about security — who use noreply addresses, who sign their commits, who set up pre-commit hooks — miss things on the first pass. Sometimes the second. The history accumulates quietly while you are focused on building. The audit is what catches what everything else missed.

---

## Why "I Deleted the File" Is Never Enough

This is the most important concept in this entire section, so it comes first.

When you delete a file and commit the deletion, git records that the file no longer exists in the current state of the repository. But every previous commit where the file existed is still fully intact in the history. Anyone who clones your repo gets the entire history — including every commit where that file was present, with all its contents.

This means:

```
You commit a file containing your API key.
You realize the mistake.
You delete the file and commit the deletion.
You push.

The API key is still in your history.
Anyone who clones the repo can run:
git show <commit-hash>
...and see the original file with the key in it.
```

The only way to actually remove something from git history is to ***rewrite*** the history — replace or remove the offending content from every commit where it appeared, and force push the rewritten history. That process is covered in full later in this section.

This also means that making a repo private after something has been exposed does not undo the exposure. Search engines crawl public repos. Automated tools mirror public repos. Forks made before you made it private still have the full history. If a secret was ever in a public repo — even briefly — treat it as compromised regardless of what you do afterward.

!!! warning "GitHub's cache is not instant"
    Even after you rewrite history and force push, GitHub caches old commit views. Someone who has a cached URL can still see the old content for some time. Requesting a cache purge from GitHub support is a separate step from rewriting history — both are required for a complete cleanup. This is covered in [After the Rewrite](#after-the-rewrite).

---

## What Is Visible in Your Public History

Before you can audit effectively, you need to know exactly what you are looking for. Here is everything that lives in your git history and is visible to anyone who can access your repository.

**Commit author information**

Every commit records the name and email address of the person who made it. This is set in your git config. If you ever committed with your real personal email address — even once, even years ago — that email is in the history of every repo you committed to with that config.

```
git log --format="%H %ae %an"
```

This shows every commit hash, email, and author name in the repo. Run it and read every line.

**File paths in deleted files**

If a file ever existed at a path like `/Users/yourusername/projects/...` and was committed — that path is in the history even if the file was deleted. File paths can reveal your real username, your machine name, your directory structure, and sometimes your employer or project names.

**Secrets in file contents**

API keys, passwords, tokens, database URLs, encryption keys — anything that was ever in a committed file is in the history. This includes files that were deleted, files that were renamed, and files whose contents changed. The original content of every version of every file is preserved.

**Commit messages**

Commit messages are part of the permanent record. If you ever wrote a commit message containing a ticket number from your employer's internal system, a reference to an internal tool or codebase, a real name, or any other identifying information — it is there.

**Branch names in merge commits**

When branches are merged, the merge commit often records the branch name. Branch names like `fix/johns-macbook-issue` or `feature/clientname-integration` can reveal information you did not intend to make public.

**Timestamps**

Every commit has a timestamp. A consistent pattern of commits at 9am-5pm in a specific timezone, combined with other signals, can reveal where you are located and potentially who you work for.

**Connected accounts**

If multiple GitHub accounts have committed to the same repo — your personal account, a work account, a client account — all of them are visible in the history. This can link identities you intended to keep separate.

---

## The Full Audit — Running the Checks

Work through these in order. Do not skip any.

**1. Audit all commit emails and author names**

```
git log --format="%ae" | sort | uniq
```

This lists every unique email address that has ever made a commit. Every single one should be a GitHub noreply address. If you see a personal email, a work email, or anything that is not a noreply address — it needs to be rewritten.

```
git log --format="%an" | sort | uniq
```

This lists every unique author name. Check for real names, usernames you no longer use, anything you do not want publicly associated with this repo.

**2. Search history for sensitive strings**

```
git log --all -p | grep -i "password\|api_key\|secret\|token\|private_key\|passphrase" | head -50
```

This searches every commit's diff for common credential-related terms. It is not exhaustive but catches the most obvious patterns.

For a more thorough scan, use truffleHog — a tool specifically designed to find secrets in git history:

```
macOS / Linux:
pip install trufflehog --break-system-packages

Run against your repo:
trufflehog git file://. --only-verified
```

truffleHog scans every commit in your history looking for high-entropy strings and known secret patterns. It is significantly more thorough than a grep search.

**3. Search for real file paths and usernames**

```
git log --all -p | grep -i "/Users/\|/home/\|C:\\Users\\" | head -50
```

This finds any commit that contains file paths referencing real user directories. If you see your actual username in any path — that commit contains identifying information.

Also search for your old handle, your real name, your machine names, and any other identifiers you want to keep out of public history:

```
git log --all -p | grep -i "yourrealname\|oldhandle\|machinename" | head -50
```

Replace the search terms with your actual identifiers.

**4. Check for wrong-account commits**

```
git log --format="%H %ae" | grep -v "noreply.github.com"
```

This shows every commit that was NOT made with a noreply address. Any commit here was made with either a personal email or a work email — neither of which should be in your public history.

If you see commits from an account you did not intend to use — a work account, an old account, a client account — those commits link that identity to this repo in the public record.

**5. Run detect-secrets against full history**

```
detect-secrets scan --all-files
```

This scans your current working tree. For a full history scan, combine with git stash to check historical states, or use truffleHog which handles history scanning natively.

**6. Check connected and linked accounts**

Go to GitHub and check which accounts have contributed to the repo:

```
Your repo → Insights → Contributors
```

Every account listed here is publicly associated with this repo. If an account appears that you did not intend to associate — its commits need to be rewritten or the account needs to be removed from the contributor history.

**7. Audit branch names**

```
git branch -a
git log --all --format="%D" | tr ',' '\n' | sort | uniq
```

Review all branch names — local and remote — for identifying information.

---

## What Can Be Made Private and How

**Making a repo private**

Making a repo private hides it from public view going forward. It does not:
- Remove anything from the history
- Undo any exposure that already happened
- Prevent people who already cloned it from having the full history
- Purge GitHub's cache of previously public content

Private is a useful setting for repos that should not be public. It is not a cleanup tool for repos that were public and contained sensitive information.

**Removing a file from history — git filter-repo**

git filter-repo is the current recommended tool for rewriting git history. It replaces the older `git filter-branch` which is slow, error-prone, and explicitly deprecated by git itself.

Install git filter-repo:

```
macOS / Linux:
pip install git-filter-repo --break-system-packages

Windows:
pip install git-filter-repo
```

**Remove a specific file from all history:**

```
git filter-repo --path path/to/sensitive-file --invert-paths
```

This removes the file from every commit in the history where it appeared. The file no longer exists in any commit — past or present.

**Replace a sensitive string throughout all history:**

Create a file called `replacements.txt`:

```
your-api-key==>REDACTED
your-real-email@example.com==>REDACTED
yourrealname==>REDACTED
yourmachinename==>REDACTED
```

Then run:

```
git filter-repo --replace-text replacements.txt
```

Every occurrence of every string in that file is replaced throughout the entire history — in file contents, commit messages, and author information.

**Rewrite commit author email across all history:**

```
git filter-repo --email-callback '
    return email if email != b"old@email.com" else b"123456789+username@users.noreply.github.com"
'
```

Replace `old@email.com` with the email you want to remove, and the noreply address with your actual GitHub noreply address.

**Rewrite both email and name in one pass:**

```
git filter-repo \
  --name-callback 'return b"yourusername" if name == b"Your Real Name" else name' \
  --email-callback 'return b"noreply@users.noreply.github.com" if email == b"real@email.com" else email'
```

**Multiple operations at once:**

git filter-repo supports combining operations in a single pass. Running it once with all your replacements is faster and cleaner than running it multiple times.

!!! warning "git filter-repo rewrites your local repo"
    After running git filter-repo, your local repository history has been rewritten. The remote (GitHub) still has the old history until you force push. Do not push anything between running filter-repo and force pushing — the histories will be incompatible and git will refuse the push without --force.

---

## After the Rewrite

Rewriting history locally is step one. These steps complete the cleanup.

**Step 1 — Force push the clean history:**

```
git push origin main --force
```

This replaces the remote history with your rewritten local history.

**Step 2 — Request a GitHub cache purge:**

GitHub caches old commit views. Even after force pushing, someone with a cached URL may still see old content. Contact GitHub support and request a cache purge:

```
https://support.github.com
Subject: Request cache purge for repository history rewrite
```

Include your repository URL and explain that you have rewritten history to remove sensitive content.

**Step 3 — Notify collaborators:**

Anyone who has cloned or forked the repo now has a history that diverges from the rewritten one. They need to:

```
# Delete their local clone and re-clone from the updated remote
# OR
git fetch origin
git reset --hard origin/main
```

Their local history still contains the old commits. Simply pulling will not fix this — they need to reset to the rewritten remote history.

**Step 4 — Address forks:**

Forks made before the rewrite still have the old history. You cannot force-rewrite someone else's fork. If the exposed information is sensitive — a real credential, personal identifying information — contact GitHub support about the specific forks and explain the situation.

**Step 5 — Revoke any exposed credentials:**

History rewriting removes the evidence. It does not undo the exposure. If an API key, password, or token was ever in a public repo — even briefly — revoke it at the source and generate a new one. Assume it was seen.

---

## If Something Went Public — Emergency Response

If sensitive information was in a public repo and you just discovered it — act in this order, without delay.

```
Step 1 — Revoke the exposed credential immediately.
  Go to the service that issued it and invalidate it now.
  Do not wait until after you have cleaned the history.
  The credential is already compromised. Revocation stops
  the bleeding while you work on the cleanup.

Step 2 — Make the repo private if it is currently public.
  This does not fix the exposure but it stops new eyes
  from finding it while you work.

Step 3 — Identify exactly what was exposed.
  Run the full audit above. Know every commit,
  every file, every string that needs to be addressed.

Step 4 — Rewrite history using git filter-repo.
  Address everything in one pass if possible.

Step 5 — Make the repo public again if appropriate.
  Only after confirming the history is clean.

Step 6 — Force push and request GitHub cache purge.

Step 7 — Generate new credentials for everything revoked.

Step 8 — Audit all systems that used the exposed credentials
  for unauthorized activity.

Step 9 — Notify affected parties if applicable.
  If user data or third-party credentials were exposed,
  you may have legal notification obligations depending
  on your jurisdiction and what was exposed.
```

***Speed matters at every step.*** Automated scanners index public repositories continuously. The window between exposure and harvest can be minutes. Revoke first, clean second, notify third.

---

## The Full Audit Checklist

Run this on every repo before calling it clean. Run it again any time you add a new machine, a new collaborator, or inherit a repo from someone else.

```
EMAILS AND IDENTITIES
□ git log --format="%ae" | sort | uniq
  → Every email is a GitHub noreply address
  → No personal emails, no work emails, no old emails

□ git log --format="%an" | sort | uniq  
  → No real names
  → No old handles
  → No machine names

□ git log --all -p | grep -i "/Users/\|/home/\|C:\\Users\\"
  → No real usernames in file paths

SECRETS AND SENSITIVE DATA
□ trufflehog git file://. --only-verified
  → No verified secrets found in history

□ detect-secrets scan --all-files
  → No unacknowledged secrets in current files

□ git log --all -p | grep -i "password\|api_key\|secret\|token"
  → No credential patterns in history

ACCOUNTS AND ACCESS
□ Repo → Insights → Contributors
  → Every contributing account is intentional
  → No wrong-account commits

□ git log --all --format="%D" | tr ',' '\n' | sort | uniq
  → No identifying information in branch names

AFTER ANY REWRITE
□ Force pushed to remote
□ GitHub cache purge requested
□ Collaborators notified and reset
□ Forks addressed if sensitive data was involved
□ All exposed credentials revoked and replaced
□ New credentials tested and working
```

---

## Prevention — So You Never Need This Section Again

The audit fixes the past. These habits prevent the future.

**Before every first commit on any machine:**

```
git config user.email
git config user.name
git config commit.gpgsign
```

Verify all three return the correct values before you touch anything. This takes ten seconds. It prevents hours of history rewriting.

**Before committing to any repo you have not touched in a while:**

```
git config user.email
```

Confirm you are still configured correctly. Configuration can drift when you work across multiple machines or accounts.

**After setting up on a new machine:**

Follow the full SSH setup in [SSH Keys →](../setup/ssh.md) before cloning anything. Configure your `~/.ssh/config` for multiple accounts before you accidentally commit as the wrong one. Verify git config before the first commit.

**The multiple account habit:**

If you have more than one GitHub account — personal, work, client, anything — configure them in `~/.ssh/config` as separate hosts before you touch any repos. Then verify which account you are using before every first commit in a new repo. The moment you commit as the wrong account, the cleanup becomes a history rewrite. The prevention is a five-second config check.

**After any collaborator leaves:**

Run the email audit. Confirm their commits are attributed correctly and that no residual access or identity information remains that creates an unwanted link.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
