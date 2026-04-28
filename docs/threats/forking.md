# Forking & Attribution

You made your code public because you wanted it out in the world — accessible, useful, free. Most developers publish with good intent: solve a problem, help someone, contribute something real. What happens to it after that — who takes it, who profits from it, who strips your name off it and calls it theirs — that's where it gets complicated. Forking is not theft. But it can be used like it.

---

## What Forking Actually Is

A fork is a copy of your repository under someone else's GitHub account. One click. GitHub makes it that easy by design — it's the foundation of how open source collaboration works. Someone forks your repo, makes changes, and optionally submits those changes back to you as a pull request. That's the intended flow.

What people often don't realize is that a fork is independent the moment it's created. The person who forked your repo can take it in any direction they want — with or without your knowledge, with or without crediting you, and with or without ever submitting anything back. What they're legally allowed to do with it depends entirely on one thing: your license.

---

## What Your License Actually Controls

Your license is the legal contract between you and everyone who uses, copies, or modifies your code. If you don't have one, that contract doesn't exist — and the default position is worse than most people expect.

**No license** does not mean free to use. In most jurisdictions, no license means all rights reserved by default. Technically, no one has the right to fork, copy, or distribute your code at all. In practice this is rarely enforced and widely ignored — but it also means you have no clear legal standing when someone does take it, because the terms were never established.

**MIT** is the most permissive common license. Anyone can use, copy, modify, and distribute your code — including in commercial products — with one requirement: they must include your original copyright notice and license text. That's it. They don't have to credit you prominently. They don't have to share their changes. They just have to keep your name in the license file that most users never read.

**Apache 2.0** is similar to MIT but adds explicit patent protection — contributors grant users rights to any patents related to their contribution. Better for projects where patent risk is a concern. Same attribution requirement as MIT.

**GPL (General Public License)** is the strongest copyleft license. Anyone who distributes a modified version of your code must also release their modifications under the GPL. This is what forces the code to stay open. If someone builds a product on your GPL code, they cannot keep their changes proprietary. This is the license that has teeth.

**Creative Commons** licenses are designed for content, not code. Don't use them for software.

> 💡 **Choosing a license**
> If you want maximum adoption with minimal friction: MIT. If you want to ensure your work stays open and any improvements get shared back with the community: GPL. If you're not sure: MIT is the safer default for most projects. Adding a license is a one-file commit. Not having one is a decision you'll regret when it matters.

### Protecting Your Work from Commercial Exploitation

Publishing your code for free doesn't mean publishing it for profit — someone else's profit. If you want to control what others can build on top of your work and whether they can monetize it, your license is the only tool you have. Here's what actually gives you protection.

**Strongest license:** GPL is the strongest for keeping code open, but if you want to stop commercial use entirely, you need to go further. AGPL (Affero GPL) is the most aggressive open source license — it closes the "SaaS loophole" where companies run your code as a service without releasing their changes. Even GPL doesn't cover that.

**For personal use only / no commercial use:** That's not actually an open source license — open source by definition allows commercial use. What you want is called a **Source Available** license. The most common ones:

- **Business Source License (BUSL)** — allows personal and non-commercial use, blocks commercial use until a set date (usually 4 years) when it converts to open source
- **Commons Clause** — an addendum you attach to any license (MIT + Commons Clause, Apache + Commons Clause) that explicitly prohibits selling the software or selling a service built on it
- **Creative Commons NonCommercial (CC BY-NC)** — technically for content not code, but some developers use it. Not ideal for software.

If you want "use it, learn from it, build on it — but don't profit from my work without talking to me first," the cleanest option is **MIT + Commons Clause**. It's readable, it's enforceable, and it's explicit.

> ⚠️ **ALWAYS license your project. EVERY TIME.**
> No exceptions. No "I'll add it later." A repo without a license is a repo with no rules — and when someone takes your work, you'll have nothing to stand on. One file. One commit. Do it when you create the repo.

To add a license to an existing repo:

```bash
# GitHub makes this easy — go to your repo and click "Add file"
# then type LICENSE — GitHub will offer you a license picker
# Or create it manually:
touch LICENSE
# Paste your chosen license text, commit, and push
git add LICENSE
git commit -S -m "docs: add MIT license"
git push origin main
```

---

## What Forking Does NOT Protect You From

Even with a solid license, there are things you cannot fully prevent.

Someone can fork your repo, rename it, remove your name from the README and documentation, and publish it as their own project. If your license requires attribution, they're violating it — but enforcement is entirely on you. GitHub will not do it for you. There is no automated system that checks.

In recent years, developers have discovered their open source tools republished on npm, PyPI, and the Chrome Web Store with zero credit — sometimes packaged and sold as paid products. The code was identical or minimally modified. The original author's name was gone. The new "author" was collecting stars, users, and in some cases money.

What happened when they found out:

- Filed DMCA takedowns where the platform supported it
- Opened issues on the infringing repo (often ignored or deleted)
- Posted publicly, which sometimes created enough community pressure to force attribution or removal
- In a small number of cases, pursued legal action — expensive, slow, and only viable when the financial harm was significant

What they mostly couldn't do: get GitHub or npm to act quickly, recover lost reputation or users, or undo the damage to their project's discoverability once a competing version existed.

> ⚠️ **Attribution is your responsibility**
> If attribution matters to you, choose a license that requires it, state it clearly in your README, and document your original publication date. That paper trail is what you have when someone takes your work.

---

## When Forking Becomes a Threat Vector

Not every malicious fork is about credit. Some are about reaching your users.

**Typosquatting forks** — A fork published under a name one character off from yours. `requests` becomes `requets`. `flask` becomes `flaask`. Users searching for your package install the wrong one. The malicious version looks legitimate, functions mostly correctly, and quietly does something else in the background.

**Backdoored forks designed to look like the original** — A fork that mimics your repo's appearance, README, and documentation but contains modified code. If your project has any visibility, attackers will use your reputation as bait. Users who find the fork through search results, a compromised link, or a redirect install something that was built to compromise them.

**Dependency hijacking through forks** — If another project depends on yours and an attacker can get their fork listed as the dependency source — through a compromised package registry account, a typosquatted package name, or a malicious PR to the dependent project — your users are now running attacker code without knowing it.

This connects directly to the next section. Supply chain attacks often start with a fork.

---

## What You Can Actually Control

You cannot prevent forking on a public repo — nor should you want to. But you can make your project harder to impersonate and easier to verify.

**Choose and display your license prominently.** A LICENSE file in the root of your repo and a license badge in your README makes the terms clear and creates a legal record. It also signals that you're serious about your project.

**State attribution requirements explicitly in your README.** Don't rely on users reading the LICENSE file. If you require credit, say so directly:

```markdown
## License

MIT License — see [LICENSE](LICENSE) for details.
If you use this project, attribution is required per the license terms.
```

**Use GitHub's fork network.** Go to your repo's Insights tab → Forks. You can see every public fork. It won't catch everything, but it lets you monitor what's out there and spot anything suspicious.

**Publish to package registries yourself, early.** If your project is a library or tool, claim your name on npm, PyPI, or whatever registry applies before someone else does. A typosquatter can't take a name that's already yours.

**Sign your releases.** Signed tags and releases give users a way to verify they're getting code that came from you. See [SSH Keys](../setup/ssh.md) for how commit signing works.

**Document your original publication date and history.** Your commit history is timestamped. Your first public release is on record. If someone tries to claim priority, your git history is your evidence.

> ⚠️ **If you find an infringing fork**
> Document everything first — screenshots, URLs, timestamps. Then: file a DMCA takedown through GitHub's process at `github.com/contact/dmca` if the license is being violated. Report to the relevant package registry if it's been published there. Post publicly if platform action is slow — community pressure works. Consult an attorney if financial harm is involved.

---

## Where This Goes Next

A malicious fork is often the first step in a supply chain attack — not just against your users, but against every project that depends on yours.

[→ Supply Chain Security](supply-chain.md) — How attackers move through the dependency graph and what you can do to stop them.

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
