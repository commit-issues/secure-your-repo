# Securing Your GitHub Account

> Do this once — before anything else.

---

!!! abstract "TL;DR"
    - Create a dedicated email for GitHub — not your personal, not your work address.
    - Use a passkey if available. If not, use an authenticator app. Never SMS only.
    - Do not use Google Authenticator — if you lose your phone, you lose your codes.
    - Save your backup codes offline the moment you enable 2FA.
    - Set your commit email to your GitHub noreply address before your first commit.
    - Enable Vigilant Mode so unsigned commits are visibly flagged.
    - Audit SSH keys and third-party app access — revoke anything you don't recognize.
    - If your account has public visibility, you are a higher-value target. Act like it.

    Already locked down? Jump to [Starting a New Repository](../setup/new-repo.md)

---

Your GitHub account is the front door to everything you build. Every repo, every SSH key, every secret stored in Actions, every deployment pipeline — it all lives behind that one login. This is not something you set up per project. You do it once, you do it right, and every project you ever build benefits from it automatically.

---

## 📧 Start With a Dedicated Email

Before you create or harden your GitHub account, set it up with an email address that exists only for this purpose. Not your personal email. Not your work email. A clean, separate address that is not connected to your real name, your employer, or any other platform you use.

- Use a provider with strong security defaults — Proton Mail is a solid choice.
- Do not use this email for anything else — no newsletters, no signups, nothing.
- Secure the email account itself with 2FA before using it for GitHub.
- If your GitHub account gets targeted, this email being separate means the attacker cannot pivot to your personal inbox.

!!! warning "If you skip this"
    Your personal email tied to GitHub means one phishing email, one breach of any service that shares that address, and an attacker has a direct path to your account. Separation is the defense.

---

## 🔐 Two-Factor Authentication — Do It Right

Enable 2FA immediately. But how you enable it matters as much as whether you enable it.

```
1. Passkey (strongest)
   A hardware key or device biometric. Phishing-resistant.
   GitHub supports passkeys — use one if you can.

2. Authenticator app (strong — recommended minimum)
   Generates time-based codes on your phone.
   NOT Google Authenticator — see note below.

3. SMS (weak — avoid if possible)
   Vulnerable to SIM swapping.
   Do not rely on this alone.
```

!!! warning "Why not SMS?"
    SIM swapping is an attack where someone convinces your carrier to transfer your phone number to a SIM they control. Once they have your number, they receive your SMS codes. This attack is well-documented and is used specifically against developers and creators with public profiles. The more visible your account, the more attractive the target.

!!! warning "Why not Google Authenticator?"
    Google Authenticator has no encrypted backup or export. If you lose or break your phone, your codes are gone and you are locked out of every account that uses it. Use an app that supports encrypted backup — Aegis on Android, Ente Auth or Raivo on iOS.

```
Where to enable:
GitHub → Settings → Password and authentication
→ Two-factor authentication → Enable
```

!!! tip "New to authenticator apps?"
    An authenticator app generates a new 6-digit code every 30 seconds. When you log in, GitHub asks for that code alongside your password. Even if someone has your password, they cannot log in without the current code from your physical device.

---

## 🔑 Save Your Backup Codes — Right Now

The moment you enable 2FA, GitHub gives you a set of one-time backup codes. These are your way back in if you ever lose access to your authenticator app.

- Download them immediately — do not skip this step.
- Store them offline — printed and locked away, or in an encrypted password manager.
- Do not store them in your email, notes app, or anywhere cloud-synced.
- Treat each code like a physical key to your account — because that is exactly what it is.

!!! warning "If you skip this"
    Lose your phone with no backup codes and you are locked out of your own account. GitHub account recovery is not instant — depending on your account history and verification options, it can take days or fail entirely.

---

## 📧 Commit Email — Mask Your Identity

Every commit you push includes the email address set in your local git config. If that is your real email, it is now permanently embedded in your commit history on every public repo you touch — visible to anyone, forever.

Switch to your GitHub noreply address before you make your first commit.

```
Step 1 — Enable email privacy on GitHub:
GitHub → Settings → Emails
→ Check "Keep my email address private"
→ Check "Block command line pushes that expose my email"

Your noreply address will look like:
123456789+yourusername@users.noreply.github.com

Step 2 — Set it in git on your machine:

macOS / Linux:
git config --global user.email "123456789+yourusername@users.noreply.github.com"

Windows:
git config --global user.email "123456789+yourusername@users.noreply.github.com"

Step 3 — Confirm it was applied:
git config user.email
```

!!! warning "If you skip this"
    Your personal email in public commit history is a permanent, harvestable data point. It can be used to find your other accounts, correlate your identity across platforms, and target you directly. Removing it retroactively requires rewriting git history across every affected repo — covered in the OSINT & Identity Leakage section.

!!! info "Already committed with your real email?"
    Fix the config now so the damage stops here. Then head to the OSINT & Identity Leakage section for the full history rewrite process using git filter-repo.

---

## 👁 Vigilant Mode

Enable Vigilant Mode so every commit on your account displays its verification status. Any commit that is not cryptographically signed — including anything pushed while impersonating you — will show as unverified.

```
Where to enable:
GitHub → Settings → SSH and GPG keys
→ Vigilant mode → "Flag unsigned commits as unverified"
```

!!! tip "What does signing a commit mean?"
    Signing a commit uses your SSH or GPG key to cryptographically prove that a specific commit came from you and was not tampered with after the fact. GitHub shows a green "Verified" badge on signed commits. We cover setting this up in the Starting a New Repository section.

---

## 🔗 Audit SSH Keys and Third-Party Access

Any SSH key registered to your account can push code to your repos. Any third-party app you have ever authorized still has that access unless you explicitly revoked it. Neither expires automatically.

```
Audit SSH keys:
GitHub → Settings → SSH and GPG keys

For each key ask:
→ Do I recognize this key name?
→ Is this machine still in my possession?
→ Has this key been rotated in the last 12 months?
Remove anything you cannot confirm.

Audit third-party apps:
GitHub → Settings → Applications → Authorized OAuth Apps
GitHub → Settings → Applications → Authorized GitHub Apps

For each app ask:
→ Do I still actively use this?
→ Does it need the level of access it was granted?
→ Is this app still actively maintained?
Revoke anything unnecessary.
```

!!! warning "If you skip this"
    A third-party app with repo access that gets compromised or sold has access to your code. You will not be notified. The access does not expire on its own.

---

## ✅ Verification Checklist

New account or existing — run through this before moving on.

```
□ GitHub account uses a dedicated, purpose-only email address
□ That email account is also secured with 2FA
□ Passkey enabled — OR authenticator app enabled (not SMS only)
□ Authenticator app supports encrypted backup (not Google Auth)
□ Backup codes downloaded and stored offline
□ Commit email set to GitHub noreply address
□ "Block command line pushes that expose my email" enabled
□ Vigilant Mode enabled
□ SSH keys audited — nothing unrecognized or stale
□ Third-party app access reviewed — nothing unnecessary
□ Password is strong, unique, stored in a password manager
□ Account recovery options do not use your personal phone number
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
