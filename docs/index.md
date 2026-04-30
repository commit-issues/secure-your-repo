---
hide:
  - navigation
  - toc
  - title
---

# &nbsp;

<div class="hero-section">
  <div class="hero-logo">
    <svg viewBox="0 0 480 310" xmlns="http://www.w3.org/2000/svg" class="main-logo">
      <defs>
        <linearGradient id="ms1" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#1e3a6e"/>
          <stop offset="100%" style="stop-color:#7c3aed"/>
        </linearGradient>
        <linearGradient id="ms2" x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" style="stop-color:#2a4f9e"/>
          <stop offset="100%" style="stop-color:#9b5de5"/>
        </linearGradient>
        <linearGradient id="mhl" x1="0%" y1="0%" x2="0%" y2="100%">
          <stop offset="0%" style="stop-color:#1e3a6e"/>
          <stop offset="100%" style="stop-color:#0f1f3d"/>
        </linearGradient>
      </defs>
      <g transform="translate(120, 0)">
      <text x="20" y="88" font-family="Georgia,serif" font-size="72" font-weight="900" fill="none" stroke="white" stroke-width="3.5" stroke-linejoin="round" letter-spacing="-2">Secure</text>
      <text x="20" y="88" font-family="Georgia,serif" font-size="72" font-weight="900" fill="#1e3a6e" letter-spacing="-2">Secure</text>
      <text x="60" y="162" font-family="Georgia,serif" font-size="72" font-weight="900" fill="none" stroke="white" stroke-width="3.5" stroke-linejoin="round" letter-spacing="-2">Your</text>
      <text x="60" y="162" font-family="Georgia,serif" font-size="72" font-weight="900" fill="#2a4f9e" letter-spacing="-2">Your</text>
      <text x="100" y="236" font-family="Georgia,serif" font-size="72" font-weight="900" fill="none" stroke="white" stroke-width="3.5" stroke-linejoin="round" letter-spacing="-2">Shi</text>
      <text x="100" y="236" font-family="Georgia,serif" font-size="72" font-weight="900" fill="#7c3aed" letter-spacing="-2">Shi</text>
      <rect x="240" y="138" width="4" height="130" fill="none" stroke="white" stroke-width="2.5" rx="1.5"/>
      <rect x="240" y="138" width="4" height="130" fill="#1e3a6e" rx="1.5"/>
      <path d="M242 145 Q301 170 301 220 Q301 258 244 270 Z" fill="none" stroke="white" stroke-width="2" stroke-linejoin="round"/>
      <path d="M242 145 Q301 170 301 220 Q301 258 244 270 Z" fill="url(#ms1)"/>
      <path d="M242 158 Q211 194 242 266 Z" fill="none" stroke="white" stroke-width="2" stroke-linejoin="round"/>
      <path d="M242 158 Q211 194 242 266 Z" fill="url(#ms2)" opacity="0.85"/>
      <path d="M244 156 Q293 178 293 220 Q293 250 246 262" fill="none" stroke="#ffffff" stroke-width="1" opacity="0.18"/>
      <path d="M201 274 Q221 266 242 269 Q263 266 283 274 Q289 284 242 291 Q195 284 201 274 Z" fill="none" stroke="white" stroke-width="2" stroke-linejoin="round"/>
      <path d="M201 274 Q221 266 242 269 Q263 266 283 274 Q289 284 242 291 Q195 284 201 274 Z" fill="url(#mhl)"/>
      <path d="M209 276 Q242 269 275 276" fill="none" stroke="#4a6fa5" stroke-width="1.2" opacity="0.45"/>
      <path d="M191 296 Q219 290 242 293 Q266 290 293 296" fill="none" stroke="white" stroke-width="1.5" stroke-linecap="round" opacity="0.3"/>
      <path d="M191 296 Q219 290 242 293 Q266 290 293 296" fill="none" stroke="#7c3aed" stroke-width="2" stroke-linecap="round"/>
      <path d="M181 308 Q199 302 217 308 Q235 314 253 308 Q271 302 289 308 Q303 312 313 308" fill="none" stroke="#9b5de5" stroke-width="1.5" stroke-linecap="round" opacity="0.7"/>
      <path d="M185 318 Q202 313 220 318 Q238 323 256 318 Q274 313 292 318" fill="none" stroke="#7c3aed" stroke-width="1" stroke-linecap="round" opacity="0.35"/>
      <text x="20" y="338" font-family="Georgia,serif" font-size="11" fill="white" font-style="italic" letter-spacing="1.2" opacity="0.5">A Full Security Engineering Field Guide</text>
      </g>
    </svg>
  </div>

  <div class="hero-sidebar" style="flex: 1; border-left: 2px solid rgba(124,58,237,0.4); padding-left: 40px;">
    <p class="hero-quote">"The guide I needed when I started — and couldn't find. Built from real knowledge, written for real people, open to the world."</p>
    <p class="hero-attribution">— SudoChef</p>
    <div class="hero-ctas">
      <a href="setup/account/" class="cta-primary">Start Reading →</a>
      <a href="https://github.com/commit-issues/secure-your-repo" class="cta-secondary">Download the Guide</a>
    </div>
  </div>
</div>

<div class="byline-strip">
  <span class="byline-label">By:</span>
  <span class="byline-name">SudoChef</span>
  <span class="byline-divider">|</span>
  <span class="byline-gh">GH: commit-issues</span>
  <span class="byline-divider">|</span>
  <a href="https://github.com/commit-issues/secure-your-repo" target="_blank" class="star-badge">
    <span class="star-icon">★</span>
    <span class="star-text">Star on GitHub</span>
    <span class="star-count" id="gh-star-count">—</span>
  </a>
</div>

<script>
fetch('https://api.github.com/repos/commit-issues/secure-your-repo')
  .then(r => r.json())
  .then(d => {
    const el = document.getElementById('gh-star-count');
    if (el && d.stargazers_count !== undefined) {
      el.textContent = d.stargazers_count.toLocaleString();
    }
  })
  .catch(() => {});
</script>

---

<div class="story-section">
<p><em>Nobody handed me a roadmap. I found my way into tech from a kitchen — no CS degree, no connections, no one telling me where to start. I googled everything. I found pieces, fragments, half-answers scattered across forums and YouTube tabs and Stack Overflow threads that assumed I already knew what I didn't. Nothing clicked. Sound familiar?</em></p>
<p><em>Well I kept going — engineering courses, AI training, more programs — I even graduated a cybersecurity program. And after all of it — I still felt like I didn't fully know what I was doing.</em></p>
<p><em>The knowledge was there — scattered across bookmarks, notebooks, Notion docs, Obsidian vaults, Slack threads, emails, and textbooks. Everything I learned lived somewhere different. Nothing connected.</em></p>
<p><em>Every single day I advise people on where to start in security, how to break into tech, where to even begin. I watch them get overwhelmed the same way I did — confused, unsure, not knowing what they don't know. Called skids for trying to learn. So I decided to do something about it. I gathered my messy notes, organized them, structured a guide, rewrote it hundreds of times until the words were exactly right — then I put it on the internet. For free.</em></p>
<p><em>Because the gap I kept seeing wasn't just "people don't know security" — it was that nobody was teaching developers to build securely from the start. Not how to prevent. Not after the breach. Not how to build securely. Not one single line. So this guide became what nobody built for us — a field guide for shipping securely, written by someone who had to learn it the hard way so you don't have to. Secure your ship — before you ever leave the dock.</em></p>
<p class="story-final"><em>If it helps even one person, every hour it took will have been worth it.</em></p>
</div>

---

<div class="audience-section">
<p class="section-label gold">Who This Is For</p>
<div class="audience-grid">
  <div class="audience-card">
    <div class="card-title pink">Developers shipping their first project</div>
    <div class="card-body">You built something. You want to ship it right. This tells you how.</div>
  </div>
  <div class="audience-card">
    <div class="card-title pink">Self-taught devs & bootcamp grads</div>
    <div class="card-body">You learned to build. Nobody gave you the cybersecurity education that should have come with it.</div>
  </div>
  <div class="audience-card">
    <div class="card-title pink">CTF players going real-world</div>
    <div class="card-body">You know how to attack. Now learn how to defend what you build.</div>
  </div>
  <div class="audience-card">
    <div class="card-title pink">Content creators & indie hackers</div>
    <div class="card-body">Building in public is great. Leaking your credentials in public is not.</div>
  </div>
  <div class="audience-card">
    <div class="card-title pink">Students & new hires</div>
    <div class="card-body">Start your career already knowing this stuff. You're in the right place.</div>
  </div>
  <div class="audience-card">
    <div class="card-title pink">Experienced devs with gaps</div>
    <div class="card-body">This will find the things you've been assuming were fine.</div>
  </div>
</div>
</div>

---

<div class="learn-section">
<p class="section-label gold">What You'll Actually Learn</p>
<div class="learn-grid">
  <div class="learn-item purple-border">
    <div class="learn-title">How to set up GitHub the right way — from day one</div>
    <div class="learn-body">SSH keys, commit signing, 2FA, and the account settings most people never touch.</div>
  </div>
  <div class="learn-item purple-border">
    <div class="learn-title">How to harden a repo so it fights back</div>
    <div class="learn-body">Branch protection, rulesets, Advanced Security, Dependabot, secret scanning. Most people leave these off.</div>
  </div>
  <div class="learn-item gold-border">
    <div class="learn-title">How to write code that doesn't betray you</div>
    <div class="learn-body">Credentials, input validation, dependencies, networking, databases, logging, and vibe coding pitfalls.</div>
  </div>
  <div class="learn-item gold-border">
    <div class="learn-title">How attackers think — and how to think like them</div>
    <div class="learn-body">OSINT, AI-assisted attacks, supply chain, forking, visibility. The threat landscape from the adversary's side.</div>
  </div>
  <div class="learn-item pink-border">
    <div class="learn-title">How to keep it secure after you ship</div>
    <div class="learn-body">Freshness, backups, email security, security debt, cron hardening, dependency intelligence, notifications.</div>
  </div>
  <div class="learn-item pink-border">
    <div class="learn-title">Things nobody talks about — but should</div>
    <div class="learn-body">Licensing, IP protection, identity leakage, legal options, business privacy. The stuff between the lines.</div>
  </div>
</div>
</div>

---

<div class="credibility-section">
<p class="section-label purple">Trusted by the Community</p>
<p class="credibility-quote">"This is the guide we've been waiting for."</p>
<p class="credibility-body">Read and validated by security professionals, educators, and community leaders across the industry. Shared with teams, recommended to interns and new hires, used in classrooms and communities. Peer reviewed and earned their stamp of approval.</p>
<div class="credibility-tags">
  <span>Security Professionals</span>
  <span>Educators</span>
  <span>Community Leaders</span>
  <span>Students</span>
</div>
</div>

---

<div class="about-section">
  <div class="about-avatar">SC</div>
  <div class="about-content">
    <p class="about-name">SudoChef</p>
    <p class="about-title">Cybersecurity · AI · Tech Educator · @sudochef</p>
    <p class="about-bio">Making security knowledge accessible to everyone — regardless of how you got here, what you studied, or where you're starting from.</p>
  </div>
</div>

---

<div class="footer-cta">
  <p class="footer-title">Ready to actually secure your shi<span class="footer-p">p</span>?</p>
  <div class="footer-btns">
    <a href="setup/account/" class="cta-primary">Begin with Setup →</a>
    <a href="https://github.com/commit-issues/secure-your-repo" class="cta-secondary">Download the Guide</a>
  </div>
</div>

<style>
/* HERO */
.hero-section {
  display: flex;
  align-items: center;
  gap: 48px;
  padding: 56px 0 48px;
}
.hero-logo { flex: 1; display: flex; justify-content: center; max-width: 560px; }
.main-logo { width: 100%; max-width: 560px; }
.hero-sidebar {
  flex: 1;
  border-left: 2px solid rgba(124,58,237,0.4);
  padding-left: 40px;
}
.hero-quote {
  font-size: 1.1rem;
  line-height: 1.9;
  font-style: italic;
  margin: 0 0 16px;
  opacity: 0.85;
}
.hero-attribution {
  font-size: 0.85rem;
  color: #9b5de5;
  letter-spacing: 1px;
  margin: 0 0 28px;
}
.hero-ctas { display: flex; gap: 12px; flex-wrap: wrap; }

/* BYLINE */
.byline-strip {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 14px 0;
  border-top: 1px solid rgba(255,255,255,0.06);
  border-bottom: 1px solid rgba(255,255,255,0.06);
  margin-bottom: 2rem;
}
.byline-label { font-size: 0.75rem; opacity: 0.35; letter-spacing: 1px; }
.byline-name { font-size: 0.75rem; color: #9b5de5; font-weight: 700; letter-spacing: 1px; }
.byline-divider { font-size: 0.75rem; opacity: 0.2; }
.byline-gh { font-size: 0.75rem; opacity: 0.35; letter-spacing: 1px; }

/* STAR BADGE */
.star-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 6px;
  padding: 4px 10px;
  text-decoration: none !important;
  transition: border-color 0.2s, background 0.2s;
}
.star-badge:hover { background: rgba(255,255,255,0.09); border-color: rgba(201,168,76,0.5); }
.star-icon { color: #c9a84c; font-size: 13px; }
.star-text { font-size: 11px; color: rgba(255,255,255,0.6); letter-spacing: 0.5px; }
.star-count { font-size: 11px; color: #9b5de5; font-weight: 700; min-width: 12px; }

/* CTAs */
.cta-primary {
  background: #7c3aed;
  color: white !important;
  padding: 11px 26px;
  border-radius: 6px;
  text-decoration: none !important;
  font-size: 0.85rem;
  font-weight: 700;
  display: inline-block;
  transition: background 0.2s;
}
.cta-primary:hover { background: #6d28d9; }
.cta-secondary {
  border: 1.5px solid rgba(255,255,255,0.25);
  color: rgba(255,255,255,0.7) !important;
  padding: 11px 26px;
  border-radius: 6px;
  text-decoration: none !important;
  font-size: 0.85rem;
  font-weight: 700;
  display: inline-block;
}

/* STORY */
.story-section {
  max-width: 780px;
  margin: 0 auto;
  font-size: 1.05rem;
  line-height: 1.9;
  opacity: 0.85;
}

/* SECTION LABELS */
.section-label {
  font-size: 0.7rem;
  letter-spacing: 3px;
  text-transform: uppercase;
  font-weight: 400;
  text-align: center;
  margin: 0 0 32px;
}
.section-label.gold { color: #c9a84c; }
.section-label.purple { color: #9b5de5; }

.story-section p { font-size: 0.9rem; line-height: 1.9; opacity: 0.85; margin: 0 0 1.2rem; }
.story-final { color: #9b5de5 !important; opacity: 1 !important; }

/* AUDIENCE */
.audience-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}
.audience-card {
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 10px;
  padding: 20px;
  transition: border-color 0.2s, transform 0.2s;
}
.audience-card:hover {
  border-color: rgba(232,121,160,0.3);
  transform: translateY(-2px);
}
.card-title { font-size: 0.85rem; font-weight: 700; margin-bottom: 8px; }
.card-title.pink { color: #e879a0; }
.card-body { font-size: 0.8rem; opacity: 0.45; line-height: 1.6; }

/* LEARN */
.learn-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}
.learn-item { padding: 0 0 0 20px; }
.purple-border { border-left: 3px solid #7c3aed; }
.gold-border { border-left: 3px solid #c9a84c; }
.pink-border { border-left: 3px solid #e879a0; }
.learn-title { font-size: 0.85rem; font-weight: 700; color: white; margin-bottom: 8px; }
.learn-body { font-size: 0.8rem; opacity: 0.45; line-height: 1.6; }

/* CREDIBILITY */
.credibility-section { text-align: center; max-width: 680px; margin: 0 auto; }
.credibility-quote {
  font-size: 1.4rem;
  font-style: italic;
  margin: 0 0 16px;
  line-height: 1.6;
}
.credibility-body {
  font-size: 0.9rem;
  opacity: 0.45;
  line-height: 1.9;
  margin: 0 0 32px;
}
.credibility-tags {
  display: flex;
  gap: 32px;
  justify-content: center;
  flex-wrap: wrap;
  font-size: 0.7rem;
  opacity: 0.25;
  letter-spacing: 1.5px;
  text-transform: uppercase;
}

/* ABOUT */
.about-section {
  display: flex;
  align-items: center;
  gap: 32px;
  max-width: 680px;
  margin: 0 auto;
}
.about-avatar {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: linear-gradient(135deg, #1e3a6e, #7c3aed);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  font-size: 1.2rem;
  font-weight: 900;
  color: white;
}
.about-name { font-size: 0.95rem; font-weight: 700; margin: 0 0 6px; }
.about-title { font-size: 0.75rem; opacity: 0.45; margin: 0 0 10px; line-height: 1.6; }
.about-bio { font-size: 0.75rem; opacity: 0.35; margin: 0; line-height: 1.6; font-style: italic; }

/* FOOTER CTA */
.footer-cta { text-align: center; padding: 3rem 0; }
.footer-title { font-size: 1.75rem; font-weight: 900; margin: 0 0 10px; letter-spacing: -0.5px; }
.footer-p { color: #7c3aed; font-style: italic; }
.footer-btns { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; }
</style>
