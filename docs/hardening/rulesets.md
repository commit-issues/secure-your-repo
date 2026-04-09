# Branch Protection Rulesets

> Main is where your project comes to life. Don't let it become a liability.

---

!!! abstract "TL;DR"
    - Create a ruleset targeting your default branch before you push any real code.
    - Enable Restrict deletions — no one should be able to delete main.
    - Enable Block force pushes — force pushing rewrites history and cannot be undone.
    - Enable Require signed commits — every commit must be cryptographically verified.
    - Set the bypass list to Repository admin only — and check it first every time.
    - Enforcement status must be set to Active or the rules do nothing.
    - For team projects — add Required reviewers and Required status checks.
    - Rulesets only affect future pushes — adding one to an existing repo does not rewrite history.

    Already have a ruleset? Jump to [Verifying Your Ruleset](#verifying-your-ruleset)

---

Imagine you have been building something for months. The code is solid, the history is clean, you have merged dozens of commits carefully reviewed and tested. Then someone — or something — runs a single command.

```
git push origin main --force
```

In seconds, your entire commit history is gone. Replaced. Every carefully reviewed merge, every documented decision, every signed commit — overwritten by whatever was in the force push. If other people have forked your repo or cloned it, their copies still have the old history. Your main branch now tells a completely different story than every other copy of your project in existence. There is no undo button. Git does not have one for this.

Or consider a different scenario. Your account credentials are compromised — a phishing attack, a stolen session token, a leaked personal access token. The attacker does not need to be subtle. They delete your main branch outright. No commits, no history, no recovery path unless you have a local copy or a fork somewhere that still has it.

These are not hypothetical edge cases. They are documented incidents that have happened to real projects, real developers, and real teams. In some cases the damage was recoverable. In others it was not.

Branch protection rulesets exist to make these scenarios impossible — or at minimum, require explicit human intervention to bypass. They are the guardrails that keep your main branch from being destroyed accidentally or maliciously, and they take less than five minutes to configure.

---

## Rulesets vs Branch Protection Rules

GitHub has two systems for protecting branches — the older Branch Protection Rules and the newer Rulesets. If you have used GitHub for a while you may have encountered both.

***Use Rulesets.*** They are the current system, they offer more flexibility, they support bypass lists with granular control, and they are what GitHub is actively developing and maintaining. Branch Protection Rules still work but they are the legacy approach. Everything in this section uses Rulesets.

```
Settings → Branches → Rulesets → New branch ruleset
```

---

## Creating Your Ruleset

When you open the ruleset creation screen you will see several fields. Work through them in order.

**Ruleset name**

Give it a name that tells you exactly what it does at a glance. Something like `main-protection` works perfectly. You will thank yourself later when you are auditing your settings and can immediately identify what each ruleset covers without clicking into it.

**Enforcement status**

This is the field that catches people out more than any other. A ruleset with enforcement set to anything other than ***Active*** does nothing. It is configured but not enforced. GitHub offers an "Evaluate" mode for testing rules without enforcing them, and a "Disabled" state for rules you want to keep but temporarily turn off. For your main branch protection — set it to Active from the moment you create it and leave it there.

```
Enforcement status: Active
```

**Bypass list**

This is one of the most important fields in the entire ruleset — and one of the most commonly misconfigured.

The bypass list defines who is exempt from the rules you are about to set. If you leave it empty, the rules apply to everyone including yourself. That sounds ideal in theory, but in practice it means you cannot push directly to main even for legitimate maintenance tasks unless you go through a pull request. For a solo project where you are the only contributor, that creates unnecessary friction.

The correct configuration for a personal or solo repo:

```
Bypass list: Repository admin
```

This means the rules apply to everyone — but you, as the repository admin, can bypass them when you explicitly need to. It is not a loophole. It is a deliberate escape hatch that requires conscious action rather than an accident waiting to happen.

!!! warning "Check the bypass list before anything else"
    When you open a ruleset to create or edit it, check the bypass list ***first*** — before you look at any of the rules. This is easy to forget and the consequences of getting it wrong go in both directions. An empty bypass list locks you out of direct pushes. A bypass list that is too permissive undermines the rules you are trying to enforce. Repository admin only is the right balance for most personal projects.

**Target branches**

This tells the ruleset which branches to apply to. Click "Add target" and select:

```
Include default branch
```

This targets whatever your default branch is — main, master, or anything else you have configured. Using "default branch" instead of hardcoding the name means the ruleset automatically follows if you ever rename the branch.

---

## The Rules

Now for the actual protections. Scroll down to the Rules section and enable these. Each one closes a specific attack or accident vector.

---

### Restrict Deletions

```
✅ Restrict deletions
Only allow users with bypass permission to delete matching refs.
```

This prevents anyone — including collaborators — from deleting your main branch. Without this rule, anyone with write access to the repository can delete the branch entirely with a single command. With this rule, deleting main requires bypass permission — which means it requires you to do it deliberately, not accidentally.

The scenario this protects against: a collaborator runs `git push origin --delete main` by mistake, or an attacker with compromised credentials deletes the branch as part of a destructive attack. Either way — without this rule, the branch is gone. With it, the push is rejected.

---

### Block Force Pushes

```
✅ Block force pushes
Prevent users with push access from force pushing to matching refs.
```

A force push (`git push --force`) overwrites the remote branch history with whatever is in your local branch. It does not merge, it does not check for conflicts, it does not warn you. It replaces. Everything that was on the remote branch that is not in your local copy is gone.

There are legitimate uses for force pushing — rebasing a feature branch before merging, cleaning up a messy history before it becomes part of main. But those use cases belong on feature branches, not on main. ***There is almost never a legitimate reason to force push to main.***

Blocking force pushes on main means that even if you accidentally run the command, even if a compromised token tries to run it, even if a misconfigured CI/CD pipeline attempts it — the push is rejected. The history is preserved.

!!! tip "What if I genuinely need to rewrite main's history?"
    It happens occasionally — removing a secret that was committed, fixing a fundamental structural problem in the history. The process is: use your bypass permission to temporarily allow the force push, make the change, verify the result, and then the protection immediately applies to all subsequent pushes again. The bypass is not permanent. It is a single deliberate action, not a persistent permission change.

---

### Require Signed Commits

```
✅ Require signed commits
Commits pushed to matching refs must have verified signatures.
```

We set up SSH commit signing in [SSH Keys →](../setup/ssh.md#setting-up-ssh-commit-signing) and enabled Vigilant Mode in [Securing Your GitHub Account →](../setup/account.md#vigilant-mode). This rule is the enforcement layer that makes signing mandatory at the branch level, not just a personal preference.

Without this rule, signed commits are a practice you follow voluntarily. Someone else pushing to the repo — a collaborator, or an attacker with stolen credentials — can push unsigned commits and they will be accepted. With this rule, every commit pushed to main must carry a verified cryptographic signature. No signature, no merge.

This matters for two reasons. First, it makes impersonation detectable — a commit claiming to be from you but lacking your signature is immediately visible as something different. Second, it creates an auditable chain of custody for your main branch. Every commit in the history of main is provably linked to an authenticated identity.

---

## Direct Push vs Pull Request — Understanding the Difference

This is something that trips up a lot of developers and it is worth understanding clearly before you configure it.

***Direct push*** means pushing commits straight to main from your local machine — the workflow we have been using throughout this guide. You make changes locally, commit, and push. Simple, fast, and perfectly appropriate for solo projects where you are the only contributor and you have the other protections in place.

***Pull request merge*** means all changes to main must go through a pull request first — even your own. You push to a feature branch, open a PR, review it, and merge it. This adds a review step and a paper trail for every change.

For a solo personal project, direct push with signed commits and force push protection is secure and reasonable. The protections we already configured catch the dangerous mistakes. Adding a mandatory PR process for a project where you are reviewing your own PRs adds friction without meaningful security benefit.

For a team project — the calculus changes entirely. When multiple people have push access, a mandatory PR process with required reviewers is not overhead, it is your primary defense against unreviewed code reaching main. One person's mistake gets caught by another person's eyes before it becomes everyone's problem.

**To require pull requests before merging, add this rule to your ruleset:**

```
✅ Require a pull request before merging

Under the expanded options:
  Required approvals: 1 (minimum for teams)
  Dismiss stale pull request approvals when new commits are pushed: ✅
  Require review from code owners: ✅ (if you have a CODEOWNERS file)
```

!!! warning "If you enable required PRs on a solo project"
    You will need to open a pull request for every change you make — including small fixes. Even as admin with bypass permission, bypassing this rule requires a conscious override. This is not wrong — some solo developers prefer it for the discipline and the paper trail. Just go in with eyes open about what it means for your daily workflow. If you find yourself bypassing it constantly, the rule is not serving you.

!!! tip "The 'one small fix' habit"
    One of the most common ways review discipline erodes on team projects is the "I'll just push this directly, it's just a one-line fix" pattern. It feels harmless. Over time it becomes the default. And then one day the "one-line fix" has a bug, nobody reviewed it, and it is on main. Required PRs eliminate that habit entirely — not by trusting people to do the right thing, but by making the wrong thing impossible.

---

## Required Status Checks

Status checks are automated tests or validations that must pass before a pull request can be merged. If you have a CI/CD pipeline that runs your test suite — or any other automated check — you can require it to pass on every PR before anything reaches main.

```
✅ Require status checks to pass before merging
  → Add the specific checks you want to require by name
  → Require branches to be up to date before merging
```

This guide does not configure required status checks for this specific repo because it is a documentation site without a test suite. But for any project that has automated tests, linting, security scans, or other CI checks — ***requiring them to pass before merge is one of the most valuable protections you can add.***

The pattern it enforces: broken code cannot reach main. Ever. Not even if someone is in a hurry, not even if the test failure "seems unrelated," not even if it is just one small fix. The check runs, it must pass, otherwise the merge is blocked.

!!! tip "What counts as a status check?"
    Any GitHub Actions workflow step can be configured as a required status check. Your test suite, your linting pass, your security scan, your build verification — if it runs in Actions and produces a pass/fail result, it can be required. To add it, run the check once so GitHub knows it exists, then go to your ruleset and add it by the name GitHub assigned it.

---

## Required Reviewers

For team projects, requiring a minimum number of reviewers before a PR can merge is a fundamental control. It means no single person — including the repo owner — can unilaterally push changes to main without another set of eyes.

```
✅ Require a pull request before merging
  Required approvals: 1 minimum, 2 for sensitive projects
  Dismiss stale reviews when new commits are pushed: ✅
```

The "dismiss stale reviews" setting is important and easy to overlook. Without it, a PR that receives approval and then gets additional commits pushed to it can still be merged on the strength of the original approval — even though the reviewer never saw the new commits. Enabling this setting means any new commit invalidates existing approvals and requires a fresh review.

For projects with sensitive files — workflow configurations, deployment scripts, security-critical code — consider pairing required reviewers with a CODEOWNERS file. CODEOWNERS lets you specify that changes to particular files or directories must be reviewed by specific people, regardless of who else has approved the PR. We cover CODEOWNERS in [Day One Checklist →](checklist.md).

---

## Does Adding a Ruleset Affect Existing Commits?

No. ***A ruleset only affects future pushes.*** Adding branch protection to a repo that already has commits does not rewrite, alter, or affect any existing history in any way.

What changes the moment you save the ruleset:

```
→ Future force pushes to main are rejected
→ Future unsigned commits to main are rejected
→ Future deletions of main are rejected
→ Any PR merge rules you configured now apply to new PRs
```

What does not change:

```
→ Existing commits remain exactly as they are
→ Existing history is untouched
→ No files are modified
→ No commits are altered
```

This means you can add a ruleset to an existing project at any point without any risk to what you have already built. The only thing that changes is what is allowed going forward.

---

## Verifying Your Ruleset

After saving, confirm the ruleset is active and configured correctly:

```
Settings → Branches → Rulesets
```

You should see your ruleset listed with a green Active indicator. Click into it and verify:

```
✅ Enforcement status: Active
✅ Bypass list: Repository admin
✅ Target: Default branch
✅ Restrict deletions: enabled
✅ Block force pushes: enabled
✅ Require signed commits: enabled
```

**Test it from the terminal:**

Try a force push and confirm it is rejected:

```
git push origin main --force
```

You should see:

```
remote: error: GH006: Protected branch update failed for refs/heads/main.
remote: error: Cannot force-push to a protected branch.
```

That error message is the ruleset working exactly as intended.

**Verify signed commits are required:**

Temporarily disable signing and attempt a push:

```
git config commit.gpgsign false
git commit --allow-empty -m "test: unsigned commit"
git push origin main
```

You should see a rejection. Then immediately re-enable signing:

```
git config commit.gpgsign true
```

!!! warning "Re-enable signing immediately after testing"
    Do not leave `commit.gpgsign` set to false. The moment you confirm the ruleset rejects unsigned commits, turn signing back on. Every commit to main should be signed.

---

## What About Other Branches?

The ruleset we configured targets your default branch — main. Feature branches, development branches, and other working branches are intentionally left unprotected by this ruleset.

This is the right call. Force pushing on a feature branch is sometimes exactly what you need to do — cleaning up commits before a merge, rebasing on top of recent main changes. Requiring signed commits on every branch adds friction to everyday development without meaningful security benefit.

The protection that matters is on the branch that matters — main. That is where your stable, released code lives. Everything else is working space.

If you run a larger project with multiple long-lived branches — a release branch, a staging branch — consider creating additional rulesets for those. The same principles apply: restrict deletions, block force pushes, require signed commits. The configuration is identical, just targeted at different branches.

---

## Coming Up in the Checklist

Two files that work directly with your rulesets and deserve their own treatment:

**CODEOWNERS** — defines which people must review changes to specific files or directories. Pairs with required reviewers to ensure sensitive files like workflow configs always get reviewed by the right person.

**SECURITY.md** — the file GitHub looks for when someone wants to report a vulnerability privately. We configured private vulnerability reporting in [GitHub Settings →](settings.md) — this is the companion file that tells reporters what to do and where to go.

Both are covered in [Day One Checklist →](checklist.md).

---

## The Bigger Picture

Branch protection rulesets are one of the clearest examples of security and stability reinforcing each other. The rules that protect your main branch from a malicious actor are the same rules that protect it from a bad day and a careless command.

When you have rulesets in place, you can push to your repo confidently knowing that the worst-case accidental command — the force push, the accidental deletion, the unsigned commit from a misconfigured machine — will be caught and rejected before it causes damage. That confidence is not complacency. It is exactly what a well-configured safety net is supposed to provide.

Next: [Advanced Security →](advanced-security.md)

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
