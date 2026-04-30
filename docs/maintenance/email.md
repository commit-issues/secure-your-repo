# Email Security

Your email is the highest value target attached to your developer identity. It's not that owning your email means owning everything — it means owning the recovery path to everything. Password resets, 2FA backup codes, domain registration, GitHub notifications, package registry accounts — all of it routes through that inbox. A hardened GitHub account buys you time. A compromised email gives an attacker a way to use that time against you.

We covered the foundational hardening — strong unique passwords, authenticator app 2FA, hardware keys, breach checking — in [Securing Your Account](../setup/account.md). If you haven't been through that section, start there. What's here is what that section didn't cover: what to do if your email gets compromised, and the surfaces most people overlook even after they've done the basics.

---

## If Your Email Gets Compromised

The cascade moves fast. An attacker with access to your inbox can request password resets for every service you use, intercept 2FA codes sent via email, read GitHub notification emails to map your project structure and collaborators, and impersonate you to services and registrars by sending from your address.

The lockout is the most damaging part. Once an attacker controls your email and changes the recovery options, getting back in becomes a support ticket process that can take days — during which they have uncontested access to everything downstream.

**If you suspect your email is compromised, move in this order:**

1. Get back into your email account — use recovery codes, backup email, or your provider's account recovery process
2. Change your email password immediately to something new and unique
3. Review and revoke any active sessions you don't recognize
4. Check your email forwarding rules — attackers often set up silent forwarding so they keep receiving your email even after you've recovered access
5. Change passwords on your highest-value accounts first — GitHub, your domain registrar, your package registry accounts
6. Notify collaborators if your project accounts were exposed

```
Gmail: Settings → See all settings → Forwarding and POP/IMAP
Outlook: Settings → Mail → Forwarding
Proton Mail: Settings → All settings → Privacy and security → Email forwarding
```

> ⚠️ **The forwarding rule is the hidden trap**
> Most people recover their account, change their password, and think they're done. The attacker set up a forwarding rule in the first sixty seconds. Check it every time.

---

## Alias Addresses — What Most Developers Skip

An email alias is a forwarding address that delivers to your real inbox without exposing your real address. If a service gets breached and your alias is leaked, your real address isn't exposed — and you can disable that specific alias without affecting anything else.

This matters for developers specifically because you sign up for a lot of services — package registries, hosting providers, CI tools, API services, developer forums. Each one is a breach surface. Aliases contain the blast radius.

Services worth using:

- **SimpleLogin** — open source, free tier, works with any email provider
- **Apple Hide My Email** — built in if you're in the Apple ecosystem
- **DuckDuckGo Email Protection** — free, forwards to any inbox, strips trackers
- **Fastmail** — paid, strong built-in alias support

Using aliases also tells you exactly which service leaked your address when you start getting spam — the alias name tells you the source.

---

## GitHub Notification Security

GitHub sends a lot of email. Some of it contains sensitive information about your projects, your collaborators, and your security alerts — and attackers know this.

**Security alert emails are real — and actively impersonated.** GitHub sends genuine Dependabot alerts, secret scanning alerts, and security advisory notifications to your email. Attackers send phishing emails designed to look exactly like these. Never click links in security alert emails — go directly to GitHub and check from there. See [AI-Assisted Attacks](../threats/ai-attacks.md) for how convincing these fakes have become.

**Notification emails reveal project structure.** Issue notifications, PR notifications, and commit comment emails contain repo names, branch names, and sometimes code snippets. A compromised inbox gives an attacker a map of your active work without ever touching GitHub.

Reduce what GitHub sends to only what you actually act on:

```
GitHub → Settings → Notifications
→ Review each category
→ Disable email for anything you don't act on immediately
→ Use GitHub's web interface for non-urgent notifications
```

---

## One More Thing — Your Developer Email Should Be Separate

If you're using a personal email account — one you share across personal and professional use, one you've had for years across dozens of services — as your primary developer identity, that's a risk worth addressing. A dedicated email address for your developer accounts, with its own strong password and 2FA, limits what's exposed if your personal email is ever compromised, and vice versa.

---

## Email Security Checklist

```
□ Account hardening complete — see Securing Your Account
□ Email forwarding rules checked — nothing unexpected
□ Alias addresses in use for developer service signups
□ GitHub notifications trimmed to only what you act on
□ Dedicated developer email separate from personal
□ Recovery codes stored securely — not in your inbox
```

---

## Where This Goes Next

Your email is locked down. Now let's talk about the security you've been putting off — the kind that accumulates silently until it becomes a crisis.

[→ Security Debt](debt.md) — What security debt is, how it builds up without you noticing, and how to pay it down without stopping everything else.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
