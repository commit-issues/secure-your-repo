# Now Go Build. Ship Securely.

You made it through the whole guide. That already puts you ahead of most people shipping code right now.

Don't wait for the perfect setup. Take what you learned today and apply it to something real — your current repo, your next project, the one you've been putting off. Security isn't a phase you do once before launch. It's how you build.

---

## Before You Ship Anything

Run through these before your next push:

- **Signed commits on?** `git config --global commit.gpgsign true`
- **Secrets out of your history?** Run `detect-secrets scan` or check with `git log -p`
- **Dependencies audited?** `pip-audit` or `npm audit` — do it now, not after
- **Branch protection set?** Restrict deletions, block force pushes, require signed commits

---

## What's in the Lab

Tools built with the same standards this guide teaches.

| Repo | What it does |
|------|-------------|
| [cve-security-monitor](https://github.com/commit-issues/cve-security-monitor) | Monitors NVD for new CVEs, alerts you on your desktop — no third parties, no cloud |
| [darkweb-exposure-toolkit](https://github.com/commit-issues/darkweb-exposure-toolkit) | Checks if your info is circulating in breach data — from your terminal |
| [nmap-reference](https://github.com/commit-issues/nmap-reference) | The nmap commands you actually need, organized and explained |
| [enumeration-reference](https://github.com/commit-issues/enumeration-reference) | Enumeration methodology for pentesting — beginner to advanced |
| [exploitation-reference](https://github.com/commit-issues/exploitation-reference) | Exploitation techniques, structured and documented |
| [google-dorking](https://github.com/commit-issues/google-dorking) | OSINT via search operators — what they are, how they work, how to use them |

---

## If This Helped You

Star the repo. Share it with someone building their first project or their fiftieth.

<!-- GitHub star button -->
<a class="github-button" href="https://github.com/commit-issues/secure-your-ship" data-color-scheme="no-preference: dark; light: light; dark: dark;" data-icon="octicon-star" data-size="large" data-show-count="true" aria-label="Star commit-issues/secure-your-ship on GitHub">Star</a>
<script async defer src="https://buttons.github.io/buttons.js"></script>

<div class="share-btns">
  <a href="https://twitter.com/intent/tweet?text=Just%20went%20through%20Secure%20Your%20Ship%20—%20a%20free%20security%20field%20guide%20for%20everyone%20who%20builds.&url=https://commit-issues.github.io/secure-your-ship/" target="_blank" class="share-btn twitter">𝕏 Share on X</a>
  <a href="https://www.linkedin.com/sharing/share-offsite/?url=https://commit-issues.github.io/secure-your-ship/" target="_blank" class="share-btn linkedin">in Share on LinkedIn</a>
  <button onclick="navigator.clipboard.writeText('https://commit-issues.github.io/secure-your-ship/now-go-build/').then(() => { this.textContent = '✓ Copied!'; setTimeout(() => { this.textContent = '🔗 Copy Link'; }, 2000); })" class="share-btn copy">🔗 Copy Link</button>
</div>

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
