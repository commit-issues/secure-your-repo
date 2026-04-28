# When Your Project Gets Stolen

The best time to protect your work was before you published it. The second best time is right now. Most developers don't think about theft until it happens to them — and by then they're scrambling, emotional, and making decisions without a plan. This section exists so you already have the plan. Read it before you need it. And if you're here because it already happened — good. You're in the right place. Here's how you go full force.

---

## Prevention First

Everything in this guide up to this point is prevention. Your license, your commit history, your signed releases, your documented publication dates — all of it is the paper trail that makes you credible and makes theft harder to get away with. If you skipped any of it, go back now.

The specific things that matter most before your work gets taken:

**License your repo.** This is non-negotiable. A repo without a license has no enforceable terms. If someone takes your unlicensed code and republishes it, you have significantly less legal standing than if you had a license that explicitly prohibits what they did. See [Forking & Attribution](forking.md) for which license gives you what protection.

**Document your original publication date.** Your git history is timestamped. Your first commit, your first public release, your first published package version — all of it is on record. Screenshot your first release. Archive your original repo page on the Wayback Machine (`web.archive.org`). These timestamps are your evidence of priority if someone tries to claim they built it first.

**Publish to package registries early.** If your project is a library or tool, claim your name on npm, PyPI, or whatever registry applies as soon as the project is public. A typosquatter or bad actor cannot take a name that already belongs to you.

**Watermark your documentation.** Your writing style, your examples, your specific phrasing — these are harder to scrub than code. If someone republishes your docs, your fingerprints are in the language. Keep originals.

**Enable Vigilant Mode on GitHub.** Signed commits with Vigilant Mode enabled mark every commit from your account as verified. If someone is impersonating you or claiming your commits as theirs, the signature verification is the proof that they didn't make them.

---

## The Forms It Takes

Theft doesn't always look like theft at first. Know what you're looking for:

**Code republished without attribution.** Your repo, copied wholesale or minimally modified, published under someone else's name. Your license stripped out or replaced. Your README rewritten just enough to look different. This is the most common form and the easiest to document.

**Your project rebranded and sold.** Your open source tool wrapped in a new UI, rebranded, and sold as a paid product or course. Your work generating someone else's income. This is the one your license is specifically designed to prevent — and why MIT + Commons Clause or AGPL matters more than plain MIT if commercial exploitation is a concern.

**Typosquatted packages.** A package published to npm or PyPI under a name one character off from yours, designed to intercept users who mistype your package name. The code may look like yours, function like yours, and quietly do something else entirely. Your users are the real target.

**Identity impersonation.** A fake GitHub account, a fake maintainer profile, or someone posing as you in issues, Discord, or email. They may be trying to social engineer contributors, extract credentials from your community, or damage your reputation. This is a different problem from code theft and requires a different response.

**Malicious fork designed to look like the original.** A fork that mimics your repo's appearance and documentation but contains backdoored or modified code. Distributed to your users through compromised links, misleading search results, or direct outreach pretending to be you.

---

## If It Happens — Document Everything First

Before you do anything else — before you post publicly, before you file a report, before you contact anyone — document everything. Evidence disappears. Repos get deleted. Accounts vanish. Cache clears. You need the record before they realize you've noticed.

**Screenshot everything:**
- The infringing repo, profile, or package page
- The timestamps on their commits and releases
- Any README, documentation, or code that matches yours
- Their account information — username, bio, linked accounts, follower count
- Any communications, issues, or comments

**Archive the URLs:**

Go to [web.archive.org](https://web.archive.org) and submit every relevant URL for archiving. This creates a timestamped, third-party record that cannot be altered after the fact.

```bash
# You can also use curl to trigger a Wayback Machine save
curl "https://web.archive.org/save/https://github.com/theirusername/stolen-repo"
```

**Record your own evidence of priority:**

```bash
# Show your original commit timestamps
git log --format="%H %ai %s" | head -20

# Show when your repo was first made public
# (visible in your GitHub repo's creation date — screenshot it)
```

Save everything locally. Multiple copies. Do not rely on a single location.

---

## Your Options — Platform by Platform

Once you have your documentation, here is what you can actually do.

### GitHub

**DMCA Takedown.** If someone has republished your code in violation of your license, you can file a DMCA (Digital Millennium Copyright Act) takedown request directly with GitHub. GitHub is legally required to respond.

Go to: `github.com/contact/dmca`

Your request needs to include: what was taken, where it lives now, proof that you are the original author, and a statement that the use is unauthorized. GitHub publishes all DMCA requests at `github.com/github/dmca` — your request will be public record.

Be aware: DMCA is a legal process. False claims have consequences. Make sure your claim is accurate and your evidence is solid before filing.

**Report the repository.** For impersonation or accounts pretending to be you, use GitHub's report mechanism:

```
Repo page → ··· menu → Report repository
Profile page → Block or Report → Report abuse
```

**Community pressure.** Post clearly and factually about what happened. The open source community responds to documented theft. Keep it factual — screenshots, timestamps, links. Emotional posts get dismissed; evidence gets shared. Other developers will amplify a legitimate, documented case.

### npm

If a package on npm is infringing your copyright or impersonating your package:

```
Go to: npmjs.com/support
Select: Report a vulnerability or abuse
Provide: Package name, your original package, evidence of infringement
```

For typosquatting specifically, npm has a dedicated process and has removed packages for this. Include your original package name, the infringing name, and the evidence that it's designed to intercept your users.

### PyPI

```
Go to: pypi.org/help/#feedback
Email: admin@pypi.org for urgent cases
```

PyPI also has a takedown process for packages that violate terms of service, infringe copyright, or are malicious. Document the same way — original package, infringing package, evidence.

### Other Platforms

If your documentation, course content, or written work has been republished on Medium, Substack, YouTube, Udemy, or any other platform — each has a copyright infringement reporting process. Look for "report copyright infringement" in their help center. Most major platforms comply with DMCA requests.

---

## If Someone Is Impersonating You

This is a different threat from code theft and requires a different response. Someone posing as you — on GitHub, Discord, email, or social media — can damage your reputation, deceive your community, and extract credentials or trust from people who believe they're talking to you.

**Immediate steps:**

- Screenshot everything — their profile, their messages, their activity
- Warn your community publicly and clearly. Post from your verified accounts. State explicitly: "I am only reachable at these accounts. Anyone else claiming to be me is not me."
- Report the impersonation to the platform — every major platform has an impersonation reporting process
- If they're contacting your users or contributors, reach out to those people directly through verified channels

**On GitHub specifically:**

```
Go to: github.com/contact/report-abuse
Select: Impersonation
Provide: The fake account URL, your account URL, evidence they're impersonating you
```

Enable [Vigilant Mode](../setup/account.md) if you haven't already — it marks your commits as verified and gives your community a way to distinguish your genuine commits from anything attributed to a fake account.

---

## Legal Escalation

Platform reports and DMCA takedowns handle a lot — but not everything. If the infringement is serious, financially damaging, or the platform isn't acting fast enough, here is what escalation actually looks like.

### For Websites and Domains

**ICANN UDRP (Uniform Domain-Name Dispute-Resolution Policy).** If someone registered a domain using your project name, brand, or handle to impersonate you or profit off your reputation, UDRP is the mechanism designed for exactly this. It is filed through ICANN-accredited dispute resolution providers — not a court — which makes it significantly faster and cheaper than litigation. Decisions typically come within 60 days. Go to `icann.org/resources/pages/help/dndr/udrp-en` to start.

**Your hosting provider's abuse team.** Every major host has an abuse reporting process that responds to copyright and IP violation claims — often faster than platform support. If you know where the infringing site is hosted, file there directly:

- Cloudflare: `cloudflare.com/abuse`
- AWS: `aws.amazon.com/premiumsupport/knowledge-center/report-aws-abuse`
- Google Cloud: `support.google.com/code/contact/cloud_platform_report`
- DigitalOcean: `digitalocean.com/company/contact/abuse`
- Netlify / Vercel: Each has an abuse contact in their terms of service

A valid IP complaint to the host can take a site offline even if the domain registrar doesn't act. Go for both simultaneously.

**Your domain registrar's abuse process.** If the domain itself is being used to infringe on your trademark or impersonate your project, file with the registrar in addition to ICANN. Registrars are bound by their own terms of service and ICANN agreements.

### For App Stores

**Apple App Store:** File at `reportinfringement.apple.com`. Apple accepts copyright, trademark, and impersonation claims. Include your original app name, the infringing app, and your evidence. Apple has removed apps for IP violations and takes these reports seriously.

**Google Play Store:** File at `support.google.com/googleplay/android-developer/contact/takedown`. Covers copyright, trademark, and impersonation. Same documentation requirements — original work, infringing app, evidence of priority.

**Microsoft Store, Steam, and others:** Each has a dedicated IP and copyright reporting process in their developer or publisher support documentation. Search "[platform name] report copyright infringement" for the direct link.

### Broader Legal Options

**Trademark.** If your project name, brand name, or logo is trademarked — or if you've established common law trademark through consistent, public, commercial use — you have legal grounds that go beyond copyright. Trademark claims are often more powerful than copyright claims for stopping commercial exploitation of your brand. A cease and desist letter from an attorney, citing trademark infringement, frequently moves faster than any platform process and puts the infringing party on notice that you are serious.

**Trade secret law.** If unpublished code, proprietary methodology, or confidential design was taken — not from your public repo, but from a private repo, a confidential communication, or an internal document — trade secret law may apply even without a patent. This varies significantly by jurisdiction but is worth raising with an attorney if the circumstances fit.

**Small claims court.** For financial damages under a certain threshold — which varies by state and country but is often $5,000–$10,000 in the U.S. — small claims court is accessible without an attorney and has been used successfully in IP and copyright cases. If someone is actively profiting from your work, documenting the financial harm and filing in small claims is a real option that doesn't require a large legal budget.

**Cease and desist.** Before full litigation, a formal cease and desist letter from an attorney puts the infringing party on legal notice and creates a paper trail that strengthens any subsequent case. Many infringers comply at this stage — especially smaller operators who aren't prepared for actual legal exposure. Services like Clerky, LegalZoom, or a local IP attorney can draft one for a fraction of full litigation costs.

> ⚠️ **Document before you escalate**
> Every legal option above depends on your evidence. Timestamps, screenshots, archived URLs, your original publication record, and your license terms are what make a legal claim viable. Without documentation, you have a complaint. With it, you have a case.

---

## The Reality Check

Here is the honest version of what you can expect:

**What you can likely recover:** Attribution, removal of infringing content on major platforms, public acknowledgment of what happened, community support.

**What takes longer:** Legal action, financial damages, removal from smaller or non-compliant platforms, international cases where DMCA doesn't apply.

**What you may not recover:** Users who already installed the malicious version, reputation damage in communities where the false narrative spread before you caught it, time.

This is not defeatism. It is preparation. The developers who come out of these situations with their projects intact and their reputation strengthened are the ones who had documentation ready, acted fast, stayed factual, and treated it like the operational problem it is — not an emotional one.

You built something worth stealing. That means it matters. Protect it like it does.

---

## Prevention Checklist — Before It Happens

```
□ Repo has a license — chosen deliberately, not defaulted
□ First release archived on web.archive.org
□ Package name claimed on relevant registries
□ Commits signed with Vigilant Mode enabled
□ Original publication dates documented
□ README states license terms clearly
□ Contact and attribution requirements stated explicitly
```

---

## Where This Goes Next

You've covered the threats. The last piece of the awareness section is understanding what public actually means — and what gets exposed even when you think something is private.

[→ Repo Visibility & Access](visibility.md) — What public vs. private actually means, what leaks anyway, and how to think about access tiers.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
