# Repo Visibility & Access

Public doesn't mean what most people think it means. And private doesn't mean what they think either. Most developers treat visibility like a light switch — on or off, public or private, safe or exposed. The reality is that visibility is a spectrum, and both settings leak more than you expect. Understanding exactly what each one exposes — and to whom — is the difference between a deliberate security posture and a false sense of control.

---

## What Public Actually Exposes

When you make a repo public, you're not just publishing your code. You're publishing a detailed record of everything that happened to that code — and some of it you probably didn't intend to share. Here are some of the things that become visible the moment you flip that switch — and who can use them against you.

**Your full commit history — and everything buried in it.** We've discussed commit history throughout this guide, and by now you know that for the most part it is permanently readable. You also know that deleting something in a later commit does not erase it from history. To put it plainly: if you accidentally committed a credential on Tuesday and removed it on Wednesday, anyone who knows where to look can still read that credential in Tuesday's diff. The delete is in the record. So is the original. If you haven't cleaned your history yet, the full process is covered in [OSINT & Identity Leakage](osint.md#fixing-what-youve-already-leaked).

**Your contributor graph.** GitHub's contributor graph shows every person who has committed to your repo, how many commits they made, and when. Think of it as a public org chart. For a solo developer, it maps your entire activity pattern — when you work, how often, how much. For a team, it tells an outsider exactly who has touched the most sensitive parts of the codebase, how long they've been involved, and who the key people are. Those key people become social engineering targets. An attacker who wants inside your project doesn't have to attack your code — they can just start with the person who wrote the most of it.

**Your dependency graph.** GitHub's Insights tab shows every dependency your project declares. An attacker can use this to instantly identify which known vulnerabilities apply to your project without ever cloning it.

**Your Actions workflows.** Your `.github/workflows` directory is public. Anyone can read exactly how your CI/CD pipeline works, what secrets it references by name, what permissions it requests, and what commands it runs. They can't read the secret values — but knowing the names and the pipeline structure is useful reconnaissance.

**Your issue history — including context around deleted issues.** Deleted issues leave traces in notifications, in references from other issues and PRs, and sometimes in cached or archived versions. If someone filed an issue disclosing a vulnerability and you deleted it without fixing it, that deletion is not invisible.

**Forks you cannot control.** The moment your repo is public, anyone can fork it. If you later delete your repo or make it private, existing forks remain. If you push a commit with a credential and then delete it, someone who forked between the push and the deletion has it. Public is permanent in ways that private is not.

**Your traffic and popularity signals.** Star counts, fork counts, watcher counts — all public. These signal value to attackers. A repo with 3,000 stars is a more attractive supply chain target than one with 3.

---

## What Private Doesn't Protect You From

Private repos are not a vault. They are a permission layer — and permission layers have edges.

**GitHub staff access.** GitHub's terms of service explicitly reserve the right to access private repository content for support, legal compliance, and abuse investigation purposes. Your private repo is private from the public — not from the platform.

**Collaborators you forgot about.** Every person you've ever added as a collaborator to a private repo retains access until you explicitly remove them. Former teammates, contractors, people you added for a one-time review — if you didn't revoke access, they still have it. Check your collaborator list. Right now.

```
Repo → Settings → Collaborators and teams
```

**Forks made before you switched to private.** If your repo was ever public — even briefly — any forks made during that window still exist. Making the repo private doesn't delete those forks or pull the code back. Whatever was in the repo when it was public is still out there in those forks.

**Actions logs.** GitHub Actions logs can persist after a workflow run and may contain output that shouldn't be public — printed environment variables, debug output, partial secret values that got echoed. Check your retention settings and review what your workflows actually print.

```
Repo → Settings → Actions → General → Artifact and log retention
```

**Deploy keys and PATs with lingering access.** A deploy key or Personal Access Token granted to a private repo doesn't expire on its own. If you created one for a project two years ago and forgot about it, it may still be active. Audit regularly.

```
GitHub → Settings → Developer settings → Personal access tokens
Repo → Settings → Deploy keys
```

**Metadata.** Even for private repos, GitHub exposes some metadata to authenticated users with access — repo name, creation date, language, topic tags. Plan your repo names accordingly. `super-secret-acquisition-target` is not a subtle name.

---

## Access Tiers — Who Should Have What

GitHub's access model has more granularity than most developers use. Understanding the tiers lets you apply least privilege — giving people exactly the access they need and nothing more.

**Personal repos — Collaborators.** On a personal account, you can add collaborators with read, triage, write, maintain, or admin access. Write access lets someone push directly to any branch unless your rulesets prevent it. Admin access lets them change settings, add other collaborators, and delete the repo. Be deliberate.

**Organization repos — Teams.** In a GitHub org, access is managed through teams rather than individual collaborators. This scales better and makes auditing easier — you see who's in which team rather than managing per-repo collaborator lists.

**Deploy keys.** A deploy key is an SSH key that grants access to a single repo — read-only or read-write. Use these for servers and automated systems that need repo access. Scope them to read-only unless a write operation is genuinely required. Rotate them when the system they belong to changes.

**Fine-grained Personal Access Tokens.** The newer PAT format lets you scope tokens to specific repos and specific permissions — contents read, issues write, actions read — rather than granting broad account-level access. Use fine-grained PATs for any automation that needs API access. Audit what exists and what it can do.

**GitHub Actions — GITHUB_TOKEN permissions.** The automatic token GitHub creates for each workflow run has configurable permissions. The default is broader than most workflows need. Set it explicitly:

```yaml
permissions:
  contents: read
  issues: write
  # Everything else defaults to none
```

Never give a workflow more permissions than the specific job requires.

---

## The Visibility Decision Framework

Before making a repo public — or keeping it private — ask these questions:

**Has the history been cleaned?**
No credentials, no personal information, no internal paths or machine names, no draft content you didn't intend to share. If the answer is no, the repo is not ready to be public. Clean the history first. See [OSINT & Identity Leakage](osint.md) for the full process.

**Is there anything in the Actions workflows that reveals too much?**
Secret names, internal service names, infrastructure details in comments or echo statements. Review before making public.

**Does the dependency graph reveal a vulnerability you haven't patched?**
If your public dependencies include a package with a known critical CVE that you haven't updated yet, making the repo public broadcasts that vulnerability. Patch first, publish second.

**Who currently has access and should they keep it?**
Audit collaborators, deploy keys, and PATs before changing visibility. Reduce access to what's actually needed before expanding who can see the repo.

**Are you ready for forks?**
Once public, the code can be forked immediately. Is your license in place? Is your README clear about attribution and usage terms? Is your package name claimed on relevant registries? Public is permanent.

**Does this need to be public at all right now?**
Private repos are free on GitHub. There is no urgency to make something public before it's ready. Ship when it's clean, licensed, and documented — not before.

---

## Common Mistakes

These are the visibility decisions developers most often regret:

**Making a repo public before cleaning the history.** The most common. A credential, a personal email, a machine name — whatever was in the history is now permanently accessible. Audit and clean before you flip the switch. Not after.

**Adding collaborators with admin access for convenience.** Admin access means someone can change your settings, add other collaborators, and delete your repo. Reserve it for people who genuinely need it. Most collaborators need write at most.

**Using a personal account for a project that should be org-level.** If your project grows, has multiple contributors, or represents a brand rather than a personal project, it belongs in a GitHub organization. Orgs have better access controls, audit logs, and team management. Migrating later is painful. Start in the right place.

**Leaving old deploy keys and PATs active.** Keys that aren't being used are attack surface that isn't providing value. Audit and revoke regularly. If you don't know what a key is for, revoke it.

**Switching a repo from private to public without telling your team.** If other people have push access to a private repo, they may have been committing with the assumption of privacy — personal notes in commit messages, draft content, debug output with internal details. Coordinate the visibility change. Don't surprise them.

**Assuming deleted means gone.** Deleted commits, deleted issues, deleted branches — none of these are reliably gone if the repo was ever public or if collaborators had access. The only safe assumption is that anything that was ever in the repo may exist somewhere outside your control.

---

## Visibility Audit Checklist

```
□ Commit history cleaned before going public
□ No credentials, personal info, or machine names in history
□ Actions workflows reviewed — no internal details exposed
□ Dependencies patched before public visibility
□ Collaborator list audited — access matches current need
□ Deploy keys audited — inactive keys revoked
□ PATs audited — fine-grained, scoped to minimum required
□ License in place before repo goes public
□ Package name claimed on relevant registries
□ GITHUB_TOKEN permissions set explicitly in workflows
```

---

## Where This Goes Next

You've covered the full Threats & Awareness section. The next block is about keeping everything you've built maintained — because security isn't a one-time setup. It's an ongoing practice.

[→ Maintenance: Data Freshness](../maintenance/freshness.md) — How to know when your data, dependencies, and configurations have gone stale — and what the risk of staleness actually is.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
