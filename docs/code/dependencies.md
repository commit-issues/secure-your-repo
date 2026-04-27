# Dependency & Supply Chain Security

> The code you did not write is still your responsibility.

---

!!! abstract "TL;DR"
    - Every package you install is code running with your full permissions. Vet it before you install it.
    - Supply chain attacks are the fastest growing threat vector in software security right now.
    - Pin your dependency versions. Unpinned dependencies update silently and can introduce vulnerabilities overnight.
    - Run pip-audit or npm audit before every commit. Automate it in CI so it runs without you thinking about it.
    - Set up automated daily scanning — manual checks find problems too late.
    - When a CVE drops for something you use — assess it the same day. Do not wait for your next scheduled check.
    - Unused dependencies are still a risk. Remove what you do not need.

    Already auditing regularly? Jump to [When a CVE Drops →](#when-a-dependency-has-a-cve)

---

Every library you install is code you did not write, running with your full permissions, on your machine, potentially with access to your credentials and your users' data. Most developers install packages without thinking twice about what they are actually adding to their project. This section covers what supply chain attacks are, how they happen, and how to protect yourself — before and after something goes wrong.

---

## Why People Use Dependencies

Nobody builds everything from scratch — and for most projects, that is the right call. Dependencies exist because common problems have already been solved, tested, and maintained by people who specialize in that specific problem. Why write your own HTTP request library when `requests` has been battle-tested by millions of projects? Why implement your own cryptography when `cryptography` was written by security experts?

Dependencies let you:

```
→ Move faster — use solved solutions instead of rebuilding them
→ Leverage expertise — cryptography, parsing, networking, 
  authentication are hard to get right from scratch
→ Reduce your own bug surface — well-maintained packages 
  have had their bugs found and fixed by a large community
→ Focus on what makes your project unique — not the plumbing
```

The tradeoff is trust. Every package you install is code you are responsible for but did not write. A typical Python web application pulls in 50 to 200 packages by the time you count direct dependencies and everything they depend on. You wrote maybe 10% of the code actually running in your project. The other 90% came from people you have never met, maintained by organizations you have never vetted, hosted on registries that have been targeted repeatedly by attackers.

That is not a reason to avoid dependencies. It is a reason to treat them with the same care you treat your own code.

---

## Supply Chain Attacks — How Big This Actually Is

A supply chain attack targets the tools and libraries that developers trust, rather than attacking applications directly. Instead of hacking your application, an attacker compromises something your application depends on. When you update your dependencies, you pull the attack into your own project — and potentially into every project that depends on yours.

This is not a theoretical threat. Supply chain attacks have become one of the most significant and fastest-growing threat vectors in software security. The reason is simple: one successful compromise of a widely-used package reaches thousands or millions of projects simultaneously. The economics are far better for an attacker than targeting individual applications one by one.

**2024-2026: the supply chain attack era**

The scale and sophistication of supply chain attacks has increased dramatically. Here is what has happened in just the last two years:

**XZ Utils backdoor (2024)**
A carefully orchestrated social engineering campaign over two years resulted in a backdoor being inserted into XZ Utils — a compression library present on virtually every Linux system. The attacker built trust with the maintainer community over an extended period before submitting malicious code. The backdoor was discovered just before it shipped in major Linux distributions. Had it gone undetected, it would have given attackers remote access to millions of servers worldwide. The attack was stopped by a Microsoft engineer who noticed unexpected CPU usage in a benchmark — essentially caught by accident.

**Polyfill.io compromise (2024)**
A widely-used JavaScript CDN service that provided browser compatibility code was acquired by a Chinese company and began serving malicious code to over 100,000 websites that included it. Consumers visiting those websites had malicious scripts running in their browsers without any action by the website developers. The websites had not changed — the dependency they loaded from an external CDN had. Major companies including Google, Ticketmaster, and JSTOR were affected.

**PyPI malware campaigns (2024-2025)**
Hundreds of malicious packages were uploaded to PyPI — the Python package index — using names designed to look like legitimate popular packages. Packages like `requests-darwin`, `aiohttp-socks5`, and dozens of others that looked plausible enough that developers downloaded them without checking carefully. These packages harvested credentials, established backdoors, and exfiltrated data from developer machines. The attacks targeted developers specifically — because a compromised developer machine often has access to production systems, API keys, and customer data.

**GitHub Actions supply chain attacks (2025)**
Attackers targeted GitHub Actions — the automation workflows that run in CI/CD pipelines. By compromising popular Actions or injecting malicious code into Actions pinned by tag rather than commit SHA, attackers gained access to repository secrets, deployment credentials, and the ability to push code to production systems. Organizations that had not followed SHA pinning practices were affected across industries.

**npm ongoing campaigns**
The npm ecosystem — used by virtually every JavaScript project — has faced continuous campaigns involving typosquatted packages, compromised maintainer accounts, and packages with hidden malicious payloads. In 2025 alone, thousands of malicious packages were identified and removed — but not before accumulating significant download counts.

**Axios npm package compromise (March 2026)**
Based on reports from early 2026, the biggest supply chain attack of the year so far is the Axios npm package compromise occurring on March 30–31, 2026. Axios is one of the most downloaded JavaScript libraries in existence — used by millions of projects worldwide for making HTTP requests. The attack, attributed to North Korean-linked actors, targeted the package directly, creating widespread potential risk across CI/CD pipelines and developer workstations globally.

This attack is worth connecting to what we covered in [Input Handling & Validation →](input-validation.md). In that section, the focus was on never trusting data that comes from outside your application — user input, form fields, API responses. The Axios compromise makes that principle even more critical: when a package that handles your HTTP requests is compromised, every API call your application makes becomes a potential exfiltration point. Your validation logic, your credential handling, your database queries — all of it runs in the same environment as the compromised library. A malicious package does not need to bypass your validation. It runs underneath it.

The attack also illustrates exactly why SHA pinning in GitHub Actions matters — covered in [GitHub Settings →](../hardening/settings.md). If your CI/CD pipeline pulled a fresh install of Axios during the compromise window without pinning to a known-good version, your pipeline may have been affected. Pinning, auditing, and automated scanning are not separate concerns. They are the same defense applied at different layers.

**Why open source is the primary target**

Open source software is infrastructure. It runs banks, hospitals, government systems, and critical services. It is trusted implicitly. Supply chain attacks exploit that trust — they get inside the perimeter by becoming part of the trusted supply.

The impact on consumers and clients is often invisible and delayed. A compromised developer tool does not immediately show up as a breach. It establishes access, exfiltrates slowly, or waits for a specific trigger. By the time the attack is discovered, data has been moving for months.

---

## How Packages Get Compromised

Understanding the attack vectors helps you recognize them before they affect your project.

**Maintainer account takeover**

Most open source packages are maintained by individuals with a single account protecting access to a package downloaded by millions. If that account has a weak password, no MFA, or reused credentials — it is a target. An attacker who gains access to the maintainer account can publish a new version of the package with any code they choose. PyPI and npm will serve it as a legitimate update.

**New maintainer — trusted fork attack**

A documented and increasingly common pattern: an attacker creates a GitHub account, builds a credible history of legitimate contributions over months or years, then offers to help maintain an unmaintained popular package. The original maintainer, often burnt out, transfers ownership. Shortly after, a malicious version is published. The XZ Utils attack used exactly this approach over a two-year timeline.

**Typosquatting**

Packages named almost identically to legitimate ones. `reqeusts` instead of `requests`. `python-dateutil2` instead of `python-dateutil`. `colourama` instead of `colorama`. These packages often contain working code that does exactly what the real package does — plus something extra running in the background. Developers who mistype a package name or copy a requirements file without checking install them without realizing.

**Dependency confusion**

If your organization uses internal package names and those names are also registered on public registries, a package manager configured to check public registries first may install the public (potentially malicious) version instead of your internal one. This attack has been used successfully against Microsoft, Apple, and dozens of other major organizations.

**Abandoned packages**

A package that was safe when you installed it may have gone unmaintained for years. Unmaintained means no one is reviewing new vulnerability reports, no one is applying security patches, and no one is watching if the package's dependencies become vulnerable. You are inheriting an accumulating security debt.

---

## Before You Install Anything — Vet It First

Every `pip install` and `npm install` is a trust decision. Make it consciously.

**Questions to ask before installing any package:**

```
→ Is this package actively maintained?
  Check the repository's last commit date.
  No commits in 12+ months = potential abandonment.

→ How many people maintain it?
  Single-maintainer packages are higher risk —
  one compromised account ends everything.

→ How widely used is it?
  High download counts and GitHub stars do not 
  guarantee safety, but obscure packages with 
  no community have less scrutiny.

→ Does it have a SECURITY.md or vulnerability 
  reporting process?
  If not, how would you know if it was compromised?

→ Have there been recent unusual changes?
  New maintainer? Sudden code obfuscation?
  Unexpected new permissions or network calls?

→ Does it have known CVEs?
  Check before you install, not after.

→ Do I actually need this?
  Could a built-in library do the same thing?
  Python's standard library is extensive — 
  many common tasks do not require a third-party package.
```

**Check for known CVEs before installing:**

```bash
# Install pip-audit if you do not have it
# macOS:
pip3 install pip-audit --break-system-packages

# Linux:
pip install pip-audit --break-system-packages

# Windows:
pip install pip-audit

# Check a specific package before installing
pip-audit --dry-run package-name

# After installing, audit your full environment
pip-audit
```

**Check the package on OSV (Open Source Vulnerability database):**

```
https://osv.dev/
Search by package name and ecosystem (PyPI, npm, etc.)
This database aggregates vulnerabilities from multiple sources
including NVD, GitHub Security Advisories, and others.
```

---

## Why Pinning Versions Matters

Pinning means specifying the exact version of every dependency in your project so that nothing updates without your knowledge or approval.

Without pinning, every time someone installs your project — or every time your CI/CD pipeline runs — it may install different versions of your dependencies than what you tested with. A package that was version 2.28.2 when you developed might be 2.32.0 when your pipeline deploys. That new version might have breaking changes. It might have a new vulnerability. It might behave differently in ways that are not obvious until something breaks in production.

With pinning, every install gets exactly the version you specified. Updates happen when you choose to update them — intentionally, with testing, with awareness of what changed.

**Python — requirements.txt:**

```
# Unpinned — updates silently, can introduce vulnerabilities
requests
flask
sqlalchemy

# Pinned — exact version, no surprises
requests==2.32.3
flask==3.0.3
sqlalchemy==2.0.31
```

**Generate a pinned requirements file from your current environment:**

```bash
pip freeze > requirements.txt
```

**Node — package-lock.json:**

In Node, running `npm install` generates a `package-lock.json` that locks the exact versions of every package and its dependencies. Commit this file. Never add it to .gitignore. It is your version lock.

```bash
# Install and lock
npm install

# Always commit package-lock.json
git add package-lock.json
```

---

## Auditing Your Dependencies

Auditing means checking your installed packages against databases of known vulnerabilities. Do this regularly — not just when you suspect a problem.

**pip-audit (Python)**

pip-audit checks your installed packages against the Python Packaging Advisory Database and the Open Source Vulnerability database. It tells you which packages have known vulnerabilities and what version fixes them.

```bash
# Audit everything installed in your environment
pip-audit

# Audit only what is listed in requirements.txt
pip-audit --requirement requirements.txt

# Get output in JSON format for automated processing
pip-audit --format json

# Attempt to auto-fix vulnerabilities
pip-audit --fix
```

What pip-audit output looks like:

```
Name          Version  ID                  Fix Versions
------------- -------- ------------------- ------------
requests      2.27.0   GHSA-j8r2-6x86-q33q 2.28.0
cryptography  38.0.0   GHSA-w7pp-m8wf-vj6r 39.0.1
```

Name and version of the affected package. The advisory ID — you can look this up on GitHub Security Advisories or NVD for full details. The version that fixes it.

**npm audit (Node)**

```bash
# Run in your project directory
npm audit

# Get detailed output
npm audit --verbose

# Attempt automatic fixes
npm audit fix

# Fix including breaking changes (use carefully)
npm audit fix --force
```

**Reading a CVE — what the numbers mean:**

Every CVE has a CVSS score — a number from 0 to 10 that represents severity.

```
9.0 - 10.0  Critical — fix immediately, same day
7.0 - 8.9   High — fix within 48 hours
4.0 - 6.9   Medium — fix within the week
0.1 - 3.9   Low — schedule it, do not ignore it
```

But the CVSS score alone does not tell you whether you are affected. A critical vulnerability in a package may only affect a specific function you do not use. Always read the advisory to understand:

```
→ Which versions are affected?
→ Which functionality is vulnerable?
→ Does your code use that functionality?
→ Is there a network-accessible attack vector 
  or does the attacker need local access?
→ What is the fix — a version update or a code change?
```

---

## Automated Daily Scanning — Why Manual Is Not Enough

Checking manually works until it does not. You check today, a CVE drops tomorrow, you find out in three weeks when you happen to run an audit again. By then, your application has been running with a known vulnerability for weeks. If it is in a production system, that is weeks of exposure.

Manual checking also requires you to know what to look for, where to look, and to do it consistently. In a real project under real time pressure, manual security checks are the first thing that gets skipped.

**The CVE Security Intelligence Monitor**

The CVE Security Intelligence Monitor (`commit-issues/cve-security-monitor`) was built specifically for this problem. It automates the entire workflow: pulling from the National Vulnerability Database, matching CVEs against your installed packages, finding available fixes, and sending alerts — daily, without you having to think about it.

What it does that manual checking cannot:

```
→ Pulls from NVD continuously — not just when you remember
→ Matches every CVE against your specific dependency list —
  not a generic list of popular packages
→ Finds available fixes and links to mitigation guidance
→ Sends notifications through your preferred channel —
  desktop, email, or both
→ Tracks which CVEs you have already reviewed and addressed
→ Runs on a schedule — 8am and 8pm by default, configurable
→ Self-monitors its own dependencies for vulnerabilities
→ Exports reports in CSV, JSON, and PDF formats
→ Integrates with your existing workflow via GitHub Actions
```

Instead of spending 30 minutes manually cross-referencing your requirements.txt against NVD every week — and inevitably missing something — the monitor does it automatically and tells you what needs your attention.

```
Repository: https://github.com/commit-issues/cve-security-monitor
```

**Setting up automated scanning in GitHub Actions:**

Even without the CVE monitor, you can automate pip-audit in your CI pipeline so it runs on every push and on a daily schedule:

```yaml
name: Dependency Security Scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 8 * * *'  # Daily at 8am UTC

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683

      - name: Set up Python
        uses: actions/setup-python@0b93645e9fea7318ecaed2b359559ac225c90a2b

      - name: Install dependencies
        run: pip install -r requirements.txt pip-audit --break-system-packages

      - name: Run pip-audit
        run: pip-audit --requirement requirements.txt
```

This runs automatically on every push, every pull request, and every morning. If a vulnerability is found, the workflow fails and you are notified.

**Staying current with security news — feeds and alerts:**

Whether you are a student just learning, an enthusiast following the field, or a professional securing production systems — staying current with breaking vulnerability news is part of the job. CVEs drop daily. Critical ones can go from published to actively exploited within hours.

```
For beginners and students:
→ CISA Known Exploited Vulnerabilities catalog
  https://www.cisa.gov/known-exploited-vulnerabilities-catalog
  Government-maintained list of actively exploited CVEs.
  If something is on this list, it is being used in real attacks.

→ CVE Trends — https://cvetrends.com
  Real-time view of which CVEs are getting the most attention
  on social media and security forums.

→ The Hacker News — https://thehackernews.com
  Accessible security news covering major vulnerabilities
  and breaches in plain language.

For practitioners and professionals:
→ NVD — https://nvd.nist.gov
  The authoritative source. Every published CVE with full
  technical detail, CVSS scores, and affected versions.

→ GitHub Security Advisories
  https://github.com/advisories
  Ecosystem-specific advisories with direct links to
  affected packages and fixed versions.

→ OSV — https://osv.dev
  Google's open source vulnerability database.
  Searchable by package, ecosystem, and CVE ID.

→ Snyk Vulnerability Database — https://snyk.io/vuln
  Detailed advisories with remediation guidance and
  exploit maturity ratings.

→ SANS Internet Storm Center — https://isc.sans.edu
  Daily briefings from security practitioners on
  current threat activity.

→ RSS feeds from NVD, CISA, and your package ecosystems
  Set these up in a feed reader and you get breaking
  vulnerability news the moment it is published.
```

---

## Popular Dependency Security Tools

A comparison of the major tools available, what each one does, and when to use it:

**pip-audit**
What it does: Scans Python packages against the Python Packaging Advisory Database and OSV. Fast, lightweight, runs locally or in CI. The recommended starting point for any Python project.
Free or paid: Free, open source.
Best for: Python projects of any size. Daily use and CI integration.

**npm audit**
What it does: Built into npm. Scans Node.js packages against the npm security advisory database. Run automatically when you install packages.
Free or paid: Free, built in.
Best for: Any Node.js project. Zero setup required.

**Dependabot**
What it does: GitHub-native automated dependency monitoring. Opens pull requests automatically when vulnerabilities are found. Also keeps dependencies updated to current versions even without CVEs.
Free or paid: Free for public repositories. Included in GitHub Advanced Security for private repos.
Best for: Any project hosted on GitHub. Set it up once and it runs forever.
Full setup: [Advanced Security →](../hardening/advanced-security.md)

**Snyk**
What it does: Multi-language vulnerability scanning with deeper analysis than pip-audit or npm audit. Includes license compliance checking, fix suggestions, and exploit maturity ratings — showing whether a vulnerability is being actively exploited in the wild.
Free or paid: Free tier available. Paid for advanced features and private repos.
Best for: Teams wanting more context than raw CVE listings. Particularly strong for Node.js and Docker.

**Socket.dev**
What it does: Goes beyond CVEs to detect supply chain-specific risks — packages with unusual network calls, obfuscated code, new maintainers with no history, install scripts that run code on your machine. Catches malicious packages that have not yet had a CVE published.
Free or paid: Free tier for open source. Paid for organizations.
Best for: High-security projects. The tool that catches what others miss — malicious packages that are new and have not been catalogued yet.

**OWASP Dependency-Check**
What it does: Enterprise-grade open source tool that scans project dependencies across multiple languages against NVD. Generates detailed HTML and XML reports. Integrates with build systems like Maven, Gradle, and Jenkins.
Free or paid: Free, open source.
Best for: Enterprise environments, Java projects, organizations that need detailed audit trails and reports.

**OSV Scanner**
What it does: Google's open source vulnerability scanner. Scans against the OSV database which aggregates advisories from GitHub, PyPI, npm, RubyGems, and more. Supports scanning lock files directly.
Free or paid: Free, open source.
Best for: Multi-language projects. Accurate results because it uses lock files rather than just package names.

**Trivy**
What it does: Scans containers, filesystems, and repositories for vulnerabilities in OS packages, language dependencies, and infrastructure configuration. The standard tool for Docker and Kubernetes security scanning.
Free or paid: Free, open source.
Best for: Projects that use Docker or deploy to containers. Catches OS-level vulnerabilities that language-specific tools miss.

**How the CVE Security Intelligence Monitor compares:**

The tools above each do part of the job. pip-audit finds known CVEs in your Python packages. Dependabot opens PRs. Snyk adds context. Socket catches new malicious packages.

The CVE Security Intelligence Monitor does the integration work — pulling from NVD, matching against your specific environment, tracking what you have already addressed, sending alerts on a schedule, and giving you a single view of your security posture. It is the difference between having five separate dashboards and having one that shows you what matters.

For a solo developer or small team who does not want to pay for Snyk or set up OWASP Dependency-Check, the CVE monitor provides the daily automated coverage that manual tools cannot.

```
Repository: https://github.com/commit-issues/cve-security-monitor
```

---

## Transitive Dependencies — The Hidden Layer

Your dependencies have their own dependencies. Those have their own dependencies. The full tree of code running in your project is much larger than your requirements.txt suggests.

A vulnerability in a transitive dependency — one you did not install directly — is still your problem. The vulnerable code is running in your environment with your permissions, even if you never knew you were depending on it.

**See your full dependency tree:**

```bash
# Python — show all dependencies including transitive
pip show package-name  # Shows what it depends on

# See the complete tree
pip install pipdeptree --break-system-packages
pipdeptree

# Node — show full tree
npm ls

# Show why a package is installed (what depends on it)
npm explain package-name
```

pip-audit and npm audit scan transitive dependencies automatically — you do not need to list them manually. This is one of the key reasons to use an automated tool rather than checking manually.

---

## When a Dependency Has a CVE

A CVE has been published for something you use. Here is the response process:

```
Step 1 — Assess the same day.
  Read the advisory. Understand what is vulnerable.
  Does your project use the affected functionality?
  What is the CVSS score?
  Is there a known exploit in the wild?

Step 2 — Determine your exposure.
  Which version are you running?
  Is your usage in the affected code path?
  Is the vulnerable functionality reachable from 
  user input or only from internal code?

Step 3 — Check for a fix.
  pip-audit will tell you the fixed version.
  If a fix exists — update immediately for Critical/High.
  If no fix exists — assess workarounds and mitigations
  from the advisory.

Step 4 — Update and test.
  pip install package==fixed-version
  Update requirements.txt
  Run your full test suite
  Run pip-audit again to confirm clean

Step 5 — Commit and deploy.
  git commit -S -m "deps: update package to X.X.X (CVE-XXXX-XXXXX)"
  Document that you reviewed and addressed it.

Step 6 — If no fix exists yet:
  Assess whether you can disable or avoid the vulnerable 
  functionality temporarily.
  Add a note to your security tracking with a date to recheck.
  Watch the advisory for a patch.
  Consider whether the risk is acceptable or whether the 
  dependency needs to be replaced entirely.
```

---

## Removing Unused Dependencies

An unused dependency is not neutral. It is code running in your environment that you are not using, may not be monitoring, and may not remember to update. It is attack surface with no corresponding benefit.

```bash
# Python — find potentially unused packages
pip install pip-autoremove --break-system-packages
pip-autoremove --list

# Or manually review what is installed vs what is imported
pip list

# Node — find unused dependencies
npm install -g depcheck
depcheck

# Remove a package and its unused dependencies
pip uninstall package-name
npm uninstall package-name
```

Before removing anything — verify it is truly unused. Some packages are required at runtime but not imported directly in your code. Test thoroughly after removal.

---

## The Dependency Checklist

```
BEFORE INSTALLING
□ Checked last commit date — active maintenance confirmed
□ Checked contributor count and maintainer history
□ Checked for known CVEs via pip-audit or osv.dev
□ Confirmed this package is actually necessary —
  no built-in alternative exists
□ Verified the package name is spelled correctly

AFTER INSTALLING
□ Version pinned in requirements.txt or package-lock.json
□ pip-audit run — no new vulnerabilities introduced
□ Package added to your automated scanning configuration

ON A SCHEDULE
□ pip-audit or npm audit run weekly minimum
□ Dependabot alerts reviewed within 48 hours of opening
□ Automated daily scan configured (CVE monitor or CI schedule)
□ Security news feeds checked for breaking vulnerabilities
□ Unused dependencies reviewed and removed quarterly

WHEN A CVE DROPS
□ Assessed same day — read the advisory, understand exposure
□ Fixed version identified
□ Updated and tested
□ Committed with clear message referencing the CVE
□ pip-audit run again to confirm clean
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
