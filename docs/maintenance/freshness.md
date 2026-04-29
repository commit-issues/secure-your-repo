# Data Freshness

Staleness is one of the most exploited gaps in security — not because developers don't care, but because nothing breaks until something does. By then it's too late to call it maintenance. It's an incident.

The uncomfortable truth is that a project that was fully secured six months ago may not be secured today. Software gets old. Tokens drift. Configs that made sense for your project at v1 don't necessarily make sense at v3. Attackers keep up with this. Most developers don't — not because they're careless, but because there's no alarm that goes off when something quietly becomes a liability.

This section is that alarm. Run through it regularly and you stay ahead of it. Skip it and you find out the hard way.

---

## What Goes Stale and Why It Matters

**Dependencies.** New vulnerabilities are discovered in existing packages constantly. A dependency that was clean when you installed it may have a critical CVE today. It doesn't update itself. See [Dependency Security](../code/dependencies.md) and [Dependency Intelligence](dependencies.md) for the full picture — but the short version is: if you haven't audited recently, audit now.

**Credentials and tokens.** API keys, Personal Access Tokens, deploy keys, and service account credentials don't expire unless you make them. A token you created eighteen months ago for a one-off integration may still have full access to your repo, your registry, or your cloud environment. If the project it was created for is done, the token should be too.

**GitHub Actions versions.** Actions pinned to tags (`@v3`) can silently change under you when a maintainer updates what that tag points to. Actions pinned to SHAs stay locked — but the underlying code may still have vulnerabilities that a newer version fixes. Both need periodic review. See [Advanced Security](../hardening/advanced-security.md) for pinning guidance.

**SSH keys.** Keys don't expire by default. A key you generated on an old machine, a key a former collaborator used, a key you added for a server that no longer exists — all of these may still have active access. If you can't account for a key, revoke it.

**Workflow permissions.** As your project grows, your GitHub Actions workflows may accumulate permissions that made sense for a feature you no longer use. Permissions that aren't actively needed are attack surface that isn't earning its keep.

**Your own security posture documentation.** If you have a `SECURITY.md`, a threat model, or a list of known risks — when did you last update it? A document that describes a project you've significantly changed is worse than no document at all. It creates false confidence.

---

## The Freshness Check — Run This Now

These commands give you a quick snapshot of where things stand. Run them on the repo you're most worried about first.

### Dependencies

=== "macOS / Linux / Kali"
    ```bash
    # Python projects
    pip list --outdated
    pip-audit

    # If your system uses pip3
    pip3 list --outdated
    pip3-audit

    # Node projects
    npm outdated
    npm audit
    ```

=== "Windows (WSL2)"
    ```bash
    # Run the same commands inside your WSL2 terminal
    pip list --outdated
    pip-audit

    # If your system uses pip3
    pip3 list --outdated
    pip3-audit

    # Node projects
    npm outdated
    npm audit
    ```

### Credentials and Tokens

Check your active Personal Access Tokens:

```
GitHub → Settings → Developer settings → Personal access tokens
```

For each token ask: Do I know what this is for? Is it still needed? When was it last used? GitHub shows last-used dates on fine-grained PATs — use that information.

Check deploy keys:

```
Each repo → Settings → Deploy keys
```

Remove any key you can't identify or that belongs to a system no longer in use.

### SSH Keys

```
GitHub → Settings → SSH and GPG keys
```

Review every key listed. If you don't recognize a key name or know which machine it belongs to, remove it. A key you can't account for is a key someone else might control.

### GitHub Actions

Review your workflow files for Actions pinned to tags rather than SHAs:

```bash
grep -r "uses:" .github/workflows/ | grep -v "@[a-f0-9]\{40\}"
```

Anything that shows a tag (`@v2`, `@main`, `@latest`) rather than a 40-character SHA should be reviewed and pinned. See [Advanced Security](../hardening/advanced-security.md) for how.

---

## How Often — Frequency Reference

> 💡 **These are general guidelines based on common industry practices and standards** (NIST, CIS Controls, DevSecOps frameworks). They represent the minimum acceptable baseline — not the ceiling. If you ship actively, work in security, or just take this seriously, the "Paranoid & Preventative" column is where you want to be.

| What | Industry Standard | Paranoid & Preventative |
|------|-------------------|------------------------|
| `npm audit` / `pip-audit` | Every dependency addition; minimum weekly | Every commit, every dependency addition, every deploy |
| `npm outdated` / `pip list --outdated` | Monthly | Weekly |
| Personal Access Tokens review | Monthly | Bi-weekly; rotate every 90 days max |
| Deploy keys review | Monthly | Monthly; rotate every 60 days |
| SSH keys review | Quarterly | Monthly |
| GitHub Actions versions review | Quarterly | Monthly |
| Workflow permissions audit | Quarterly | Every time a workflow changes |
| `SECURITY.md` and documentation review | Every major version | Every significant change, minimum quarterly |
| Full repo security posture review | Every 6 months | Quarterly |
| Collaborator access audit | When someone leaves | Monthly — people's roles change, access should too |
| Dependabot alerts triage | Weekly | Daily |

> 💡 **Make it a habit, not an event**
> The developers who stay ahead of staleness aren't running massive quarterly audits — they're doing small checks consistently. Five minutes after every dependency addition. A quick token review at the start of each month. It compounds over time into a project that's genuinely hard to exploit through neglect.

---

## Signs Something Has Gone Stale

Not everything shows up in an audit command. Watch for these behavioral signals:

- A dependency you rely on has gone quiet — no commits, no releases, no maintainer activity in over a year. Unmaintained packages don't get security patches.
- You get a Dependabot alert you've been dismissing. Every dismissed alert is a decision. Make sure it was a deliberate one.
- A collaborator left the project and you haven't audited access since.
- You added a token or key for a feature that got cut and never cleaned it up.
- Your GitHub Actions workflows haven't been touched in months but your project has changed significantly around them.
- You shipped a major version and never updated your `SECURITY.md`.

Any of these is a signal to run the freshness check above before moving on to the next feature.

---

## Freshness Checklist

```
□ Dependencies audited for known CVEs
□ Outdated packages identified and update plan in place
□ Personal Access Tokens reviewed — unused tokens revoked
□ Deploy keys reviewed — unrecognized keys removed
□ SSH keys reviewed — unaccountable keys removed
□ GitHub Actions pinned to SHAs, not tags
□ Workflow permissions reviewed and scoped to minimum
□ SECURITY.md and documentation current
□ Dependabot alerts reviewed — none dismissed without a reason
□ Collaborator access reflects current team
```

---

## Where This Goes Next

Freshness keeps you current. But what happens when something goes wrong despite your best efforts — when you need to go back to a known good state?

[→ Backup & Recovery](backups.md) — What a real backup strategy looks like for a GitHub-based project and how to recover from the worst.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
