# Supply Chain Security

You already know what a supply chain attack is — we covered the definition, the mechanics, and the defensive tooling in [Dependency Security](../code/dependencies.md). Now flip the camera.

That section was about defense. This one is about the adversary — how they think, how they choose targets, how they execute, and what the attack looks like from their side. Understanding the attacker's logic is what turns a checklist into an instinct.

---

## How Attackers Choose Their Targets

Supply chain attacks are not random. They are calculated. The math is simple: compromise one package that sits inside thousands of applications and you've effectively attacked all of them simultaneously — without ever touching a single one of those apps directly.

Attackers look for:

**High download volume with low maintainer bandwidth.** A package with 5 million weekly downloads maintained by one person working in their spare time is a high-value, low-resistance target. The maintainer is overwhelmed, PRs pile up unreviewed, and any attacker with patience and a halfway decent contribution history can eventually get access.

**Packages with broad permissions.** A package that runs at install time (`postinstall` scripts in npm, for example) can execute arbitrary code the moment someone runs `npm install`. No user interaction required. The install is the attack.

**Abandoned or transfer-ready packages.** When a maintainer steps away from a popular package and posts publicly that they're looking for someone to take it over, attackers watch those threads. The event-stream attack — where a malicious actor approached the original maintainer directly, offered to help maintain it, was granted publish access, and immediately injected a cryptocurrency harvester targeting a specific Bitcoin wallet — started exactly this way. The original maintainer trusted a stranger with a clean GitHub profile and polite emails. That was enough.

**Packages deep in the dependency tree.** The further down the dependency chain a package sits, the less likely anyone is watching it. You installed `requests`. `requests` depends on `urllib3`. `urllib3` depends on something else. Each layer down is one more place an attacker can hide.

**GitHub Actions with mutable tags.** An Action pinned to `@v3` is not pinned to a specific commit — it's pinned to wherever the maintainer points that tag. If an attacker compromises the maintainer's account and moves the `v3` tag to a malicious commit, every workflow using `@v3` runs attacker code on their next trigger. The tj-actions/changed-files compromise in 2025 worked exactly this way — the tag was silently moved, no workflow files were changed, and the attack was invisible until a researcher noticed unusual behavior in CI logs.

---

## The Attack Playbook

Knowing the categories is useful. Seeing how each one actually executes is what makes the threat real.

**Maintainer account takeover.** The attacker compromises the credentials of a legitimate package maintainer — through phishing, credential stuffing against a reused password, or a breach of another service where the maintainer used the same email and password. Once in, they publish a new version with malicious code. The package still works. The malicious payload runs silently alongside it. By the time anyone notices, the compromised version has been downloaded millions of times.

This is why maintainer accounts with publish access to widely used packages are high-value targets. Your npm or PyPI credentials are not just about your own projects — they're a door into every project that depends on you.

**The long game — fake contributor.** The xz utils attack is the clearest documented example. A persona named "Jia Tan" spent nearly two years making legitimate, quality contributions to the project. Patient. Helpful. Technically sharp. Built trust with maintainers, gradually took on more responsibility, eventually gained commit access. Then planted a carefully obfuscated backdoor in the compression library — one that would have granted SSH root access to any system running the affected version. It was caught by accident.

Two years. One fake identity. Nearly made it into the Linux kernel.

**Dependency confusion.** Many companies use internal package registries for private packages — libraries built in-house that aren't published publicly. Dependency confusion attacks exploit the way package managers resolve names: if an attacker publishes a public package with the same name as your internal one, and your package manager checks the public registry first, it may install the attacker's version instead of yours. This technique was demonstrated by security researcher Alex Birsan in 2021, who successfully planted packages inside Apple, Microsoft, and Uber using this method — ethically, as a proof of concept, with responsible disclosure. Attackers use the same technique without the responsible disclosure part.

**GitHub Actions pipeline compromise.** Your CI/CD pipeline runs with access to your secrets, your deployment credentials, and your production environment. A compromised Action that runs in your pipeline can exfiltrate all of it — silently, in the background, while your workflow appears to complete normally. The output in your logs looks clean. The secrets are already gone.

---

## What They're Actually After

Supply chain attackers are not always after your specific project. Often you are collateral — a path to something more valuable.

**Secrets in CI/CD environments.** Your pipeline has access to API keys, deployment tokens, cloud credentials, and signing keys. A compromised Action that runs in your workflow can read every secret you've configured and send it home. This is what the tj-actions attack was doing — dumping the runner's memory to extract credentials from environment variables.

**Credentials on developer machines.** A malicious package that executes at install time runs with your user permissions. It can read your SSH keys, your `.env` files, your shell history, your git credentials, and anything else accessible to your user account. It can do this in milliseconds, silently, before the install even finishes.

**Code execution in production.** A backdoored dependency that makes it into a production deployment gives the attacker a persistent foothold in your infrastructure. Not just access to your secrets — access to your running environment, your data, and your users.

**Your users.** If your tool or library is used by other developers or end users, a compromised version of your package is a vector into all of them. You become the unwilling distributor of the attack.

---

## The Signals You Can Actually See

Most supply chain compromises are caught late — after the malicious version has been distributed widely. But there are signals, and knowing them means you have a chance to catch things earlier.

**Unexpected new versions.** A package that releases infrequently suddenly drops a new version. No changelog. No GitHub release. No discussion in issues. A version bump with no context is worth investigating before upgrading.

**New maintainers or ownership transfers.** Package registries make ownership transfers relatively easy. If a package you depend on changes maintainers — especially if the original maintainer was active and then suddenly hands off — treat the next version with extra scrutiny.

**Install-time scripts you didn't expect.** Check for `postinstall` scripts in `package.json` for npm packages. A package that didn't previously run code at install time and now does is a red flag.

**Permission changes in GitHub Actions.** If an Action you use changes the permissions it requests, or starts accessing secrets it didn't previously touch, that change should be visible in the diff. Review it before updating.

**Version tags that move.** If you're using Actions pinned to tags rather than commit SHAs, you have no guarantee the tag points to the same commit it did yesterday. This is why SHA pinning exists. See [Advanced Security](../hardening/advanced-security.md) for how to do it.

> ⚠️ **You will not always see it coming**
> The xz utils backdoor passed code review. The event-stream attack was live for weeks before detection. The tj-actions compromise was in the wild before anyone noticed the logs. The signals help — but they are not a guarantee. Defense in depth is the only honest strategy.

---

## Your Hardening From the Attacker's Perspective

The defensive steps from [Dependency Security](../code/dependencies.md) are worth revisiting here — not as a checklist, but as the specific points where the attack stops.

**Pinning versions** stops the silent update attack. If you're locked to a specific version, a newly compromised release doesn't reach you automatically. You have to explicitly choose to upgrade — which means you have a moment to notice.

**Auditing with pip-audit or npm audit** catches known CVEs in your dependency tree. It doesn't catch zero-days or novel supply chain attacks, but it closes the most common window.

**Pinning GitHub Actions to commit SHAs** stops the mutable tag attack entirely. A SHA is a cryptographic fingerprint of a specific commit. It cannot be moved. If an attacker compromises a maintainer and moves a tag to a malicious commit, your SHA-pinned workflow is unaffected.

```yaml
# Vulnerable — tag can be moved
- uses: actions/checkout@v4

# Safe — SHA cannot be moved
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
```

**Least privilege for workflow permissions** limits the blast radius if an Action is compromised. If your workflow only has read access to contents and nothing else, a compromised Action in that workflow cannot push code, read secrets it wasn't granted, or touch your deployment environment.

**Reviewing what you actually install** — every package, every dependency — is the unglamorous work that catches things automated tools miss. Not every malicious package has a known CVE. Some are caught only because someone read the code.

---

## Where This Goes Next

You've built something. Someone wants to take it. The next section covers what happens when they do — and what your options actually are.

[→ When Your Project Gets Stolen](stolen.md) — What to do when your code, your identity, or your project is taken without your permission.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
