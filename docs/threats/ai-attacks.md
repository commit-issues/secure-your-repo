# AI-Assisted Attacks

AI didn't create attackers. It just made them faster, cheaper, and more convincing. The barrier to pulling off a sophisticated attack used to be skill. Now it's just access to a chatbot. Just like Vibe Coding is now the norm — so is Vibe Hacking.

The person targeting you doesn't need to know how to write malware. They don't need to understand network protocols or social engineering psychology. They need to know how to type a prompt. And so does everyone else.

---

## What Changed and Why It Matters

Not long ago, a convincing phishing email took time. A working exploit took skill. A fake identity that could infiltrate an open source project took months of patience and technical knowledge. These things still happen — but now they scale.

Here's how easy it actually is. Someone opens an AI tool and types:

> *"Write me a professional email pretending to be from GitHub security, telling the recipient their account has been flagged and they need to verify their SSH key by clicking this link."*

That's it. In seconds they have a polished, correctly formatted, grammatically perfect phishing email. No skill required. No practice. No experience. Just a prompt.

The same applies to malware, vulnerability scanning, fake personas, and social engineering scripts. The tooling exists, it's accessible, and people are using it against developers right now. Here are just a few examples of what that looks like in practice:

---

## AI-Generated Phishing

For years, the advice was simple: look for bad grammar, weird formatting, and suspicious links. That advice is now dangerously outdated.

AI generates phishing emails that are indistinguishable from legitimate communications. Perfect grammar. Correct tone. Accurate branding. Contextually relevant content that references your actual projects, your actual username, your actual activity — pulled from the OSINT trail you leave behind on GitHub and the web.

Developers and open source maintainers are high-value targets because they have repository access, deployment keys, and the ability to push code that reaches thousands of downstream users. In 2024, security researchers documented a significant spike in AI-crafted spear-phishing campaigns targeting maintainers specifically — emails mimicking GitHub security alerts, dependency vulnerability notifications, and CI/CD failure reports. Each one designed to create urgency and trigger a click before the target has time to think.

**What to watch for:**

- Emails creating urgency around account access, SSH keys, or secret exposure
- "Security alerts" asking you to verify credentials through a link
- GitHub notification emails that don't originate from `@github.com` or `@noreply.github.com`
- Any communication asking you to act fast on something that can wait five minutes

> ⚠️ **The urgency is the attack**
> Legitimate security systems do not require you to act in the next ten minutes or lose access. If an email is pushing you to move fast, that pressure is intentional. Slow down. Verify through a separate channel. Go directly to the platform rather than clicking the link.

---

## AI-Assisted Malware and Exploit Writing

WormGPT. FraudGPT. EscapeGPT. These are not hypothetical tools — they are real, actively sold on dark web marketplaces, and purpose-built to do what mainstream AI tools refuse to: write functional malicious code without guardrails.

Attackers use these tools to generate working malware faster than they could write it themselves, produce obfuscated payloads designed to evade detection, adapt existing public exploits to target specific versions of software, and generate variations of the same attack to avoid signature-based detection.

The output isn't always perfect. But it doesn't have to be. It just has to work once.

For developers, this means the window between a vulnerability being discovered and a working exploit existing in the wild has gotten significantly shorter. A CVE that used to take skilled attackers days to weaponize can now be turned into functional exploit code in hours — sometimes before a patch is even available.

This is one of the strongest arguments for keeping your dependencies current and your exposure minimal. The faster the exploit cycle gets, the more expensive it becomes to sit on unpatched software. See [Dependency Security](../code/dependencies.md) and [Dependency Intelligence](dependencies.md) for how to stay ahead of it.

---

## Automated Vulnerability Scanning at Scale

You don't need to be famous or valuable for this to apply to you. Automated tools don't choose targets — they scan everything.

AI-powered recon tools continuously crawl public GitHub repositories looking for exposed API keys, hardcoded credentials, outdated dependencies with known CVEs, misconfigured GitHub Actions workflows, and overly permissive repository settings. What used to require a skilled attacker spending hours on a single target now runs automatically across thousands of repos simultaneously.

The discovery-to-exploitation gap on exposed secrets is measured in minutes. Researchers have demonstrated that API keys committed to public repos are picked up by automated scanners within seconds of the push — before the developer has even closed their terminal.

> ⚠️ **"I'll fix it later" doesn't exist for secrets**
> The moment a credential is pushed to a public repo, assume it is compromised. Rotate it immediately. Not after you finish what you're working on. Now. See [Credential Management](../code/credentials.md) for the full process.

---

## Social Engineering at Scale

This is where AI-assisted attacks get genuinely sophisticated — and where the xz utils backdoor stands as the clearest warning the security community has.

In 2024, a previously unknown contributor named "Jia Tan" was discovered to have spent nearly two years building trust in the xz utils open source project. Patient, helpful, technically competent. Submitted quality contributions. Built relationships with maintainers. Gradually gained commit access. Then planted a carefully obfuscated backdoor in one of the most widely used compression libraries in Linux — one that, had it not been caught by a Microsoft engineer investigating an unrelated SSH performance anomaly, would have given attackers root-level access to countless systems.

The persona was almost certainly AI-assisted. The patience was deliberate. The target was trust.

AI makes this kind of attack cheaper to run and easier to scale. Generating a believable contributor persona — complete with consistent writing style, technical knowledge, and a history of reasonable contributions — is something a capable LLM can assist with extensively. Running multiple fake personas across multiple projects simultaneously is now operationally feasible in a way it never was before.

It also shows up in smaller, more common ways:

- **Fake security researcher outreach** — "I found a vulnerability in your repo, here's a link to the report" — designed to get you to click or hand over credentials
- **Fake job offers** targeting developers with access to sensitive codebases
- **Fake contributor accounts** submitting PRs with malicious code buried in otherwise legitimate-looking changes
- **Impersonation of known maintainers** in issues, Discord, and email

**How to think about this:**

Trust is earned through process, not personality. A contributor who is helpful, friendly, and technically sharp is not automatically trustworthy. Verify identities out of band when access is being requested. Review PRs for what the code actually does, not just whether the person seems legitimate. Be especially skeptical of anyone who pushes to accelerate the trust-building process.

> 💡 **The xz lesson**
> The attack didn't succeed through technical brilliance. It succeeded through patience and social trust. Your defenses against this aren't technical — they're procedural. Slow down. Verify. Never let urgency or social pressure shortcut your review process.

---

## What You Can Actually Do

AI-assisted attacks are a real and growing threat. They are not unstoppable. Most of the defenses are not exotic — they're the same fundamentals applied with more discipline.

**Verify out of band.** If you get an email, a DM, or a message about your account, your repo, or your credentials — don't click anything in that message. Go directly to the platform and check from there. Call or message the person through a channel you already trust if the request involves access.

**Slow down on urgency.** Urgency is a manipulation tool. Legitimate systems give you time. When something is pushing you to act in the next few minutes, that pressure is the attack.

**Review what code actually does.** Not just who submitted it. AI-generated malicious PRs can look clean at a glance. Read the diff. Understand the change. If you don't understand it, don't merge it.

**Rotate exposed secrets immediately.** No exceptions. Assume automated scanners found it before you noticed.

**Keep dependencies current.** The shorter the window between vulnerability disclosure and your patch, the smaller your exposure to AI-accelerated exploit development.

**Trust process over personality.** Friendly, helpful, technically competent — none of that means safe. Your access controls, review requirements, and signed commit policies exist precisely because trust is a social layer that can be faked.

---

## Where This Goes Next

AI-assisted attacks are often the delivery mechanism for something larger — a supply chain compromise, a credential harvest, a backdoor. The next section covers the infrastructure those attacks target.

[→ Forking & Attribution](forking.md) — What happens to your code once it leaves your hands.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
