# OSINT & Identity Leakage

You didn't just publish code. You published a trail. And someone who knows where to look can follow it straight to you.

OSINT — Open Source Intelligence — is the practice of collecting information about a target using only publicly available sources. No hacking required. No special tools. Just Google, a few databases, and patience. OSINT is legal in most jurisdictions, but it is ALMOST **NEVER** innocent.

While it is "legal" (depending on your country), it is **NOT ETHICAL** without consent. The people doing it to you without your knowledge aren't curious — they're collecting. Whether they're storing it for later or moving fast toward a target, the intent is the same: to gain leverage over someone who doesn't know they're being watched.

Don't let anyone convince you that passive information gathering is harmless. Storing someone's location, schedule, real name, employer, and social connections "just in case" is not curiosity. It is preparation. And what they're preparing for is never in your favor.

---

## What GitHub Exposes About You

Most developers treat their GitHub profile as a portfolio. It is also a data source. Here is what someone can extract from your public repos without logging in:

**Your email address.** Every Git commit contains the email address used to make it. If you haven't configured GitHub to mask your commit email, your real address is embedded in the commit history of every public repo — and that history is permanent. Even if you delete the repo, the commits may exist in forks.

Run this against any public repo to see it in action:

```bash
git log --format="%ae" | sort -u
```

That command dumps every unique commit email from the history. Anyone can run it on your repos. If you find your real email in the output, don't panic — jump to [Fixing Your Commit Email](#fixing-your-commit-email) below.

**Your username patterns.** If your GitHub handle matches your Reddit, Discord, Instagram, or email prefix, you've just connected all of those accounts together for anyone who looks. Usernames are correlation points.

**Your commit timestamps.** When you commit tells people where in the world you likely are. A developer committing consistently at 2–4am UTC is probably not in California. Commit patterns across weeks can establish time zones, work schedules, and daily routines.

**Your machine name.** If you've ever pasted terminal output into a README, an issue, a PR comment, or a commit message — and that output contained your machine hostname — you've leaked it. Hostnames are frequently real names, location hints, or organization identifiers.

**File paths.** Screenshots, error logs, and copy-pasted stack traces often include full local file paths like `/Users/yourname/projects/...`. That's your real name and confirms the type of OS system you're using. All from one pasted error message. This in the hands of the wrong person can do a lot of damage — they can find vulnerabilities and exploits specific to your OS, and combine that with any other software details you've leaked to build a targeted attack profile against you.

**Linked accounts.** Your GitHub profile bio, README, and pinned repos may link directly to your Twitter/X, LinkedIn, personal site, or email. Each one of those is another surface.

> ⚠️ **This is not theoretical**
> Security researchers have demonstrated repeatedly that a GitHub username alone is enough to find a developer's full name, employer, city, and sometimes home address — using only public information, in under 15 minutes. This is not a sophisticated attack. It requires no technical skill.

---

## Beyond GitHub — Your Domain and Website

If you've built a product, published a portfolio, or registered a domain for your project, there is another trail that has nothing to do with code.

**WHOIS lookup.** When you register a domain, the registrar is required to collect your name, address, phone number, and email. By default, most of that information is publicly searchable through WHOIS — a global database anyone can query. Go to [lookup.icann.org](https://lookup.icann.org) right now and search your own domain. What comes back may surprise you.

```bash
whois yourdomain.com
```

If you registered without privacy protection enabled, your home address may be sitting in that database. For free. Accessible to anyone.

**What to do:** Enable WHOIS privacy (also called domain privacy or proxy registration) through your registrar. Most major registrars — Namecheap, Cloudflare, Google Domains — offer this at no extra cost or for a small fee. It replaces your personal details with the registrar's proxy information.

**SSL certificate transparency logs.** Every SSL certificate issued for a domain is logged in public Certificate Transparency logs. If you've registered subdomains — `api.yourdomain.com`, `internal.yourdomain.com`, `staging.yourdomain.com` — those are discoverable even if they're not linked anywhere. Tools like [crt.sh](https://crt.sh) let anyone search them.

This is how attackers find infrastructure you thought was private.

**Your hosting provider.** An IP lookup on your domain can reveal your hosting provider, your server's approximate location, and sometimes additional domains hosted on the same IP. If you're self-hosting, that IP may narrow down your physical location significantly.

---

## If You've Built a Business

If your app, tool, or project has a business entity behind it — LLC, sole proprietorship, registered company — the public records attached to that entity are another OSINT surface.

Business registrations are public records in most U.S. states and many countries. Depending on how you registered, your personal name and address may be listed as the registered agent or business owner — searchable through your state's Secretary of State database.

**What this means in practice:**

- Someone searches your domain
- Finds your company name in the WHOIS or your site footer
- Searches your state's business registry
- Finds your home address listed as the registered agent address

That entire chain takes about four minutes.

**How to protect yourself:**

Use a **registered agent service** instead of your personal address. A registered agent is a third party authorized to receive legal documents on your behalf. Their address appears in the public record instead of yours. Services like Northwest Registered Agent, Registered Agents Inc., or your state bar's referral list typically cost $50–$150/year.

Consult a business attorney about the right entity structure for your situation. An LLC provides liability protection and, when structured correctly with a registered agent, can significantly limit what personal information appears in public filings. This is not legal advice — it's a starting point for a conversation with someone qualified to give it.

> 💡 **Privacy stacks**
> The goal is to create separation at every layer: your code commits use a masked email, your domain uses WHOIS privacy, your business filings use a registered agent, your social accounts use a consistent but intentional persona. No single layer is a guarantee. Together they make you significantly harder to profile.

---

## Audit Your Own Exposure

Before you can fix anything, you need to see what's already out there. Run this on yourself the way an adversary would.

**GitHub audit:**

```bash
# Check what email is embedded in your commits
git log --format="%ae" | sort -u

# Check what name is attached
git log --format="%an" | sort -u
```

Then check your GitHub settings:

```
GitHub → Settings → Emails
→ Ensure "Keep my email addresses private" is checked
→ Ensure "Block command line pushes that expose my email" is checked
```

If your real email is already in commit history, the commits themselves cannot be rewritten on the public record without a full history rewrite — and even then, forks may retain it. Configure the masked email going forward and accept that older repos may carry it.

**Domain audit:**

```bash
whois yourdomain.com
```

Look for your name, address, phone, and email in the output. If any of it is real, enable domain privacy through your registrar immediately.

**Certificate transparency:**

Go to [crt.sh](https://crt.sh) and search your domain. Review every subdomain listed. If anything appears that shouldn't be public-facing, investigate whether it's still live and whether it needs to be.

**Google yourself:**

```
"your full name" site:github.com
"your email" -site:yoursite.com
"your username" -site:github.com -site:instagram.com
```

Use negative operators to search outside your known accounts. What surfaces on pages you didn't put it on is the part that matters.

---

## Fixing What You've Already Leaked

If the audit above surfaced anything — your real email, your real name, a username you've since changed, a machine or device name, a hardcoded path — don't just accept it and move on. That's not good enough. You can fix it, it's not as hard as it sounds, and doing it is worth your time.

Here's the full process.

**Step 1 — Switch to your GitHub noreply address going forward.**

Go to GitHub → Settings → Emails. Check "Keep my email addresses private." GitHub will show you a noreply address in the format `username@users.noreply.github.com` — copy it.

Configure Git locally to use it:

```bash
git config --global user.email "your-noreply@users.noreply.github.com"
```

Verify it took:

```bash
git config --global user.email
```

Enable the push block so it catches you if you slip:

```
GitHub → Settings → Emails
→ Check "Block command line pushes that expose my email address"
```

**Step 2 — Audit everything, not just email.**

Before you rewrite anything, run a full scan. You're looking for all of the following embedded anywhere in your commit history:

```bash
# Real email addresses
git log --format="%ae" | sort -u

# Real names attached to commits
git log --format="%an" | sort -u

# Hardcoded usernames, machine names, or device names in file content
git log -p | grep -i "Users/yourname\|MacBook\|your-machine-name"
```

Also check for hardcoded values inside the actual files — real names in comments, device hostnames in config examples, old usernames in documentation, personal paths that slipped through:

```bash
# Search current files for anything personally identifying
grep -r "yourname\|yourmachine\|your@email.com" . --include="*.py" --include="*.md" --include="*.yml"
```

Know everything you're dealing with before you start rewriting.

**Step 3 — Back up everything first, file by file.**

Before touching a single commit, back up the entire project. Not just a zip — copy each file individually so you have a clean restore point if anything goes wrong during the rewrite.

```bash
cp -r ~/your-repo-path ~/your-repo-path-backup-$(date +%Y%m%d)
```

Verify the backup exists before continuing:

```bash
ls ~/ | grep backup
```

**Step 4 — Rewrite the commit history.**

Rewriting each commit's history is not as scary as it sounds. You've probably done it before without realizing it. The tool for this is `git filter-repo` — install it first if you don't have it:

```bash
# macOS
brew install git-filter-repo

# Linux
pip install git-filter-repo --break-system-packages
```

Then rewrite the offending email across all commits:

```bash
git filter-repo --email-callback '
    return email.replace(b"your.real@email.com", b"your-noreply@users.noreply.github.com")
'
```

To rewrite a name:

```bash
git filter-repo --name-callback '
    return name.replace(b"Your Real Name", b"sudochef")
'
```

To remove a file entirely from history (hardcoded credentials, config with personal paths, etc.):

```bash
git filter-repo --path sensitive-file.txt --invert-paths
```

**Step 5 — Force push and verify.**

After the rewrite, your local history no longer matches remote. Force push to update it:

```bash
git push origin main --force
```

Then verify the cleanup worked:

```bash
git log --format="%ae %an" | sort -u
```

Nothing personal should appear in that output.

> ⚠️ **If your repo has forks**
> Force pushing rewrites your copy of the history but does not affect existing forks — those still have the old commits. If the exposure is serious, reach out to fork owners directly. If the forks are abandoned or low-traffic, document what was exposed and when, and move forward.

> 💡 **Use this as a learning moment**
> Every time you do a history rewrite, you get cleaner about what you commit going forward. Review your templates, your README examples, your error message screenshots — anywhere a real name, machine name, or personal path could slip through. Build the habit of checking before you commit, not after.

For a full walkthrough of account-level email hardening, see [Securing Your Account](../setup/account.md).

---

## What to Lock Down — Checklist

```
□ GitHub commit email set to GitHub noreply address
□ "Block command line pushes that expose my email" enabled
□ GitHub profile bio doesn't link accounts you want separated
□ No machine names or real file paths in READMEs, issues, or comments
□ Domain registered with WHOIS privacy enabled
□ Business filings use a registered agent, not personal address
□ Subdomains audited via crt.sh
□ Usernames reviewed for cross-platform correlation
□ Google search run for your name, email, and handles
```

> ⚠️ **The permanent record problem**
> Information that has been indexed, cached, or forked cannot be fully recalled. The practical goal of OSINT hygiene is not erasure — it is making you a harder target going forward and limiting what new trails you create.

---

## Where This Goes Next

OSINT is how attackers profile you before they target you. What they do with that profile is a different problem — and it connects directly to the next section.

[→ AI-Assisted Attacks](ai-attacks.md) — How attackers use AI to weaponize the information they've collected.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
