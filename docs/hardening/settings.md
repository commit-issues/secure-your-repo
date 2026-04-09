# GitHub Repository Settings

> Most of what GitHub ships as default prioritizes convenience over security. This section walks through what to change, what to leave alone, and exactly why.

---

!!! abstract "TL;DR"
    - Disable features you are not actively using — every enabled feature is an attack surface.
    - Restrict pull request creation to prevent unsolicited contributions on repos that don't need them.
    - Lock down Actions permissions — never allow all actions from anywhere.
    - Require SHA pinning on Actions to prevent supply chain attacks via tag updates.
    - Set workflow permissions to read-only by default.
    - ***Save each section separately — Actions settings have individual Save buttons.***

    Already know the basics? Jump to [The Settings Reference](#the-settings-reference)

---

When you create a new repository on GitHub, it arrives pre-configured with a set of defaults. Most of those defaults are reasonable for a team that wants to collaborate openly with the broadest possible toolset. Most of them are not ideal if you care about security, privacy, or controlling exactly who can do what to your repository.

The settings covered in this section are not exotic hardening techniques. They are the standard repository configuration choices that experienced developers make deliberately — and that most beginners never look at because nobody told them they existed. Spending ten minutes on these settings when you create a repo saves you from a category of problems that are genuinely difficult to recover from after the fact.

This section covers General settings and Actions settings. Branch protection rulesets and Advanced Security each have their own dedicated sections — [Branch Protection →](rulesets.md) and [Advanced Security →](advanced-security.md) — because they deserve the full treatment.

---

## Understanding the Settings Page

GitHub's repository settings are found at:

```
Your repo → Settings (tab in the top navigation)
```

The settings page is long and organized into sections. We are going to work through the ones that matter most for security, in the order you will encounter them scrolling down the page. Not every setting needs to change — many of the defaults are fine. What matters is making each choice consciously rather than accepting whatever GitHub decided was convenient.

One thing worth understanding before you start: GitHub's settings UI has evolved significantly over the years and continues to change. The names, locations, and available options for certain settings may look slightly different depending on when you are reading this. The underlying concepts do not change even when the UI does. If something looks different on your screen, look for the equivalent option — the purpose is always the same.

---

## General Settings — Features

The Features section controls which GitHub features are active on your repository. Each enabled feature adds functionality — and adds surface area. Surface area that is unused is surface area that provides no value while potentially providing a vector for unwanted interaction.

```
Settings → General → scroll to Features
```

Here is what each feature does and the recommended configuration:

**Wikis**

GitHub Wikis give you a separate space for documentation attached to the repository. It is a reasonable feature for active open source projects where contributors need a collaborative knowledge base.

For most personal projects and tools — disable it. If your documentation lives in the repo itself (which it should, for version-controlled content), a separate Wiki is redundant. If you do keep it enabled, check "Restrict editing to collaborators only" — otherwise anyone with a GitHub account can edit your Wiki.

```
Recommended: Disabled for personal/solo projects
             Enabled + restricted for active open source projects
```

**Issues**

Issues are GitHub's built-in bug tracker and feature request system. They are genuinely useful for active projects where you want the public to report problems or suggest improvements.

For a tool you are building for yourself, a portfolio piece that is not accepting contributions, or any repo where you do not want unsolicited input — disable Issues. An open Issues tab on a repo you are not actively monitoring invites noise and creates an expectation of response that you may not intend to meet.

```
Recommended: Disabled if you are not actively managing contributions
             Enabled if you want public bug reports or feature requests
```

**Sponsorships**

This enables a Sponsor button on your repository profile. It is opt-in and disabled by default. Leave it disabled unless you are actively seeking sponsorship for the project.

```
Recommended: Disabled unless actively sought
```

**Discussions**

GitHub Discussions is a forum-style feature for community conversation. It is valuable for large open source projects with active communities. For most repositories it is unnecessary overhead.

```
Recommended: Disabled
```

**Projects**

GitHub Projects is a project management tool — kanban boards, roadmaps, task tracking. It is useful for teams coordinating work across multiple repos. For a solo project or a documentation repo, it adds nothing.

```
Recommended: Disabled for solo/small projects
```

**Pull Requests**

Pull Requests cannot be fully disabled on public repositories — they are fundamental to how GitHub collaboration works. What you can control is who can create them and how they are handled once they exist.

Under the Pull Requests section you will find options for merge strategy. GitHub enables all three by default — merge commits, squash merging, and rebase merging. Having all three available is not a security risk, but it is worth being intentional. Pick the merge strategy that matches how you want your history to look and disable the others.

```
For clean, linear history:      Enable squash merging only
For full commit preservation:   Enable merge commits only
For a solo project:             Any single strategy works — pick one
```

The "Pull request permissions" dropdown controls who can create pull requests. By default it is set to "All users." If your repo is not open for outside contributions, change this to restrict creation to collaborators only.

!!! warning "An open pull request is not just a suggestion"
    A pull request from an unknown account can contain malicious code, modified workflow files, or embedded instructions designed to compromise your CI/CD pipeline. Even if you never merge it, reviewing it carelessly or checking it out locally to test it can create risk. The fewer unsolicited pull requests you receive, the less exposure you have to this vector. See [AI-Assisted Attacks →](../threats/ai-attacks.md) for more on malicious pull request patterns.

**Preserve this repository**

This enrolls your repository in the GitHub Archive Program, which periodically archives public repositories for long-term preservation. It is a good thing for genuinely open source projects. For a private or sensitive project, consider whether you want your code in an external archive.

```
Recommended: Enabled for public open source projects
             Your choice for everything else
```

---

## General Settings — Commits

Scroll past Features to the Commits section.

**Require contributors to sign off on web-based commits**

This setting requires anyone making a commit through GitHub's web interface to sign off on the Developer Certificate of Origin — a statement that they have the right to contribute the code they are submitting. It is relevant for projects that have legal or compliance requirements around contribution attribution. For most personal projects it adds friction without meaningful benefit.

```
Recommended: Disabled for personal projects
             Consider enabling for projects with compliance requirements
```

**Allow comments on individual commits**

This allows anyone who can view your repository to leave comments on specific commits. It sounds harmless — and usually is — but it is another form of unsolicited interaction on your repository. If you are not running an active open source project, disable it.

```
Recommended: Disabled
```

---

## General Settings — Pushes

The Pushes section contains an option to limit how many branches and tags can be updated in a single push. At first glance this sounds like a minor operational detail — but the security reasoning behind it is significant and worth understanding.

If your GitHub account is ever compromised — through a stolen session token, a phishing attack, a leaked personal access token, or any other method — an attacker with push access can do enormous damage very quickly. A bulk push operation can simultaneously update dozens of branches, overwrite tags that point to stable releases, and inject malicious code into multiple places at once. That damage propagates instantly to anyone who has forked your repo, anyone running automated deployments tied to your branches, and any collaborator whose workflow depends on your tags being trustworthy.

Limiting bulk pushes does not prevent a determined attacker who has sustained access — but it raises the cost of a rapid, large-scale attack and gives you a smaller blast radius to recover from if something does go wrong.

This is marked as a Preview feature at the time of writing. If it is available on your account, consider enabling it with a conservative limit. If it is not yet available, note it for future review.

```
Recommended: Enable with a limit of 1-3 branches per push if available
             Monitor for availability if not yet rolled out to your account
```

---

## Actions Settings — What Is CI/CD and Why Does It Matter Here?

Before diving into the specific settings, it is worth understanding what GitHub Actions actually is and why it sits at the center of so many security conversations.

**CI/CD** stands for **Continuous Integration / Continuous Deployment**. It is the practice of automating the repetitive parts of software development — running tests every time code is pushed, building the project automatically, deploying updates to a live server without manual intervention.

- ***Continuous Integration*** means every time someone pushes code, automated tests run immediately to catch problems before they reach the main branch. No waiting, no manual test runs, no "it worked on my machine."
- ***Continuous Deployment*** means that when code passes those tests and gets merged, it is automatically deployed — to a staging environment, a production server, a documentation site, or wherever it needs to go.

GitHub Actions is GitHub's built-in CI/CD platform. You define workflows in YAML files stored in `.github/workflows/`. GitHub runs those workflows automatically based on triggers you specify — on push, on pull request, on a schedule, or manually. In this very repository, Actions is what deploys your documentation to GitHub Pages every time you push to main.

This is genuinely powerful. It is also a significant attack surface for one fundamental reason: ***workflows run code***. And that code runs with access to things that matter — your repository secrets, your deployment credentials, your API keys stored in GitHub Secrets. If an attacker can influence what code runs in your workflows — through a malicious pull request, a compromised dependency, or a supply chain attack on a third-party Action — they can potentially exfiltrate your secrets, modify your code, or use your runner as a platform for further attacks.

The settings in this section exist specifically to limit that risk. Each one closes a specific door that GitHub leaves open by default.

```
Settings → Code and automation → Actions → General
```

!!! warning "Actions settings have individual Save buttons"
    Unlike most GitHub settings pages where there is one Save button at the bottom, the Actions settings page has a separate Save button under each section. If you make a change and scroll past the Save button without clicking it, the change is lost. ***Save each section before moving to the next.***

---

## Actions Permissions

This is the most important Actions setting. It controls which Actions — the pre-built automation scripts published by GitHub, companies, and individual developers — your workflows are allowed to use.

GitHub defaults to "Allow all actions and reusable workflows." This means any Action from any author anywhere on GitHub can run in your workflows. This is the most convenient setting and the least secure.

To understand why, you need to understand how Actions are referenced in workflow files. When your workflow uses a third-party Action, it references it like this:

```yaml
- uses: actions/checkout@v4
```

That `actions/checkout` is a GitHub repository. `@v4` is a tag pointing to a specific version. Here is the problem: tags in git are movable. The owner of that repository can update the `v4` tag to point at a completely different commit at any time — including a commit that contains malicious code. If you are referencing a tag, you are trusting that the tag will always point at the same safe code. That trust can be violated.

This is called a supply chain attack, and it has happened to real, widely-used Actions. A typosquatted Action — one named almost identically to a legitimate one, like `actions/check0ut` instead of `actions/checkout` — is another vector. The name looks right at a glance. The code inside does something else entirely.

**The correct configuration:**

```
Select: "Allow [your-username], and select non-[your-username],
         actions and reusable workflows"
```

This allows Actions from your own account plus specific trusted external Actions that you explicitly permit by name. When you need a third-party Action, you add it to the allowed list deliberately — not by opening the door to everything.

**Also enable:**

```
✅ Require actions to be pinned to a full-length commit SHA
```

A commit SHA is a unique cryptographic fingerprint of a specific commit. Unlike a tag, it cannot be moved or updated. It refers to exactly one commit, forever. When you pin an Action to its full SHA instead of a tag, you are saying: "I want this exact version of this code, and I want proof that it has not changed."

!!! tip "What is a SHA and how does it relate to commit signing?"
    SHA stands for Secure Hash Algorithm. It is the same concept behind the signed commits we set up in [SSH Keys →](../setup/ssh.md#setting-up-ssh-commit-signing). When you sign a commit, GitHub generates a SHA fingerprint of that commit's contents and uses your key to prove it came from you. When you pin an Action to a SHA, you are using that same fingerprint to lock the Action to one immutable point in its history. If anything in that commit changes — even a single character — the SHA changes. You cannot fake or forge a SHA match. This is why SHA pinning is the gold standard for supply chain security in GitHub Actions.

    To find the full SHA for an Action:
    ```
    Go to the Action's GitHub repository
    → Click on the tag or release you want to use
    → Click the commit hash shown next to it
    → Copy the full 40-character SHA from the URL or commit details
    ```

    Use it in your workflow like this:
    ```yaml
    - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
    ```

    Yes, it is long. Yes, it is worth it.

---

## Fork Pull Request Workflows

This setting controls who needs approval before their pull request can trigger a workflow run.

Here is the risk it addresses. When someone external submits a pull request to your repo, GitHub can automatically run your CI/CD workflows against their code. If those workflows have access to your repository secrets — which they often do, for deployment credentials and API keys — a malicious pull request could be crafted specifically to exfiltrate those secrets during the workflow run. The attacker does not even need the pull request to be merged. The workflow run itself is the attack.

```
Select: "Require approval for all external contributors"
```

This means every pull request from someone outside your collaborators list requires your manual approval before any workflow runs. It adds one step to your review process — but it means no workflow runs on untrusted code without your explicit sign-off.

---

## Workflow Permissions

This controls the default level of access the `GITHUB_TOKEN` has when your workflows execute.

The `GITHUB_TOKEN` is an automatically generated credential that GitHub creates for each workflow run. It allows your workflows to interact with the repository — pushing commits, creating releases, posting comments on pull requests, updating branch statuses, and more. It is scoped to your repository and expires when the workflow run ends, but within that window it can do a lot.

GitHub's default in some configurations is "Read and write permissions" — meaning every workflow that runs has the ability to modify your repository by default. This is overly permissive. A workflow that only needs to read your code to run tests has no reason to have write access. Least privilege applies here exactly as it applies everywhere else in security.

```
Select: "Read repository contents and packages permissions"
```

This sets the baseline to read-only. If a specific workflow legitimately needs write access — like a deployment workflow that pushes a built site to a branch — you grant it explicitly within that workflow file rather than giving everything write access globally.

**To verify your current workflow permissions from the terminal using GitHub CLI:**

```
# Install GitHub CLI if you don't have it:
# macOS:   brew install gh
# Linux:   see https://cli.github.com/manual/installation
# Windows: winget install --id GitHub.cli

# Authenticate:
gh auth login

# Check workflow permissions for your repo:
gh api repos/yourusername/your-repo-name \
  --jq '.permissions'
```

This returns the permission levels currently set for the repository. Cross-reference what you see with what you configured in the UI.

**Also confirm:**

```
☐ Allow GitHub Actions to create and approve pull requests — DISABLED
```

Allowing Actions to approve pull requests means automated processes can approve and potentially merge code without any human review. That removes a critical checkpoint in your security posture. Keep it disabled.

---

## The Settings Reference

Here is the complete reference for every setting covered in this section. Use this for initial setup or to audit an existing repository.

```
GENERAL → FEATURES
✅ Wikis                         → Restrict editing to collaborators only
☐  Issues                        → Disabled if not managing contributions
☐  Sponsorships                  → Disabled unless actively sought
☐  Discussions                   → Disabled
☐  Projects                      → Disabled for solo projects
✅ Pull requests                  → Enabled, single merge strategy selected
✅ Restrict PR creation           → Collaborators only if not open source
✅ Preserve this repository       → Enabled for public open source

GENERAL → COMMITS
☐  Require contributor sign-off  → Disabled for personal projects
☐  Allow commit comments         → Disabled

GENERAL → PUSHES
✅ Limit bulk branch/tag updates  → Enable if available, 1-3 branch limit

ACTIONS → PERMISSIONS
✅ Allow your account + selected  → Your account + explicitly allowed only
✅ Require SHA pinning            → Always enabled
✅ Fork PR approval               → All external contributors require approval
✅ Workflow permissions           → Read only
☐  Actions create/approve PRs    → Always disabled
```

---

## What These Settings Do For You Going Forward

Once configured, most of these settings run passively. You do not think about them again — they just work.

The Actions permissions setting means that if a pull request tries to introduce a workflow using an unrecognized Action, GitHub blocks it before it runs. SHA pinning means that a trusted Action you pin today cannot silently become an untrusted one tomorrow. Read-only workflow permissions mean that even if a workflow is somehow compromised, it cannot push malicious code back to your branches or modify your releases. Fork PR approval means no workflow runs against untrusted code without your eyes on it first.

None of these make your repository impenetrable. What they do is raise the cost and complexity of attacks targeting your automation pipeline — which is exactly what hardening is supposed to accomplish.

Next: [Branch Protection →](rulesets.md)

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
