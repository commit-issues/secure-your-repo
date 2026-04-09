# SSH Keys — How Your Machine Talks to GitHub

> Set this up once per machine. Every repo you create on that machine benefits automatically.

---

!!! abstract "TL;DR"
    - Generate an ed25519 SSH key — not RSA, not ECDSA.
    - Always use a passphrase when generating your key.
    - Add the key to your SSH agent so you don't type your passphrase every time.
    - ***Only the .pub file goes to GitHub — never the private key.***
    - Test the connection before you try to push anything.
    - Configure ~/.ssh/config if you have more than one GitHub account.
    - Set up SSH commit signing so every commit gets a Verified badge.
    - If a key is ever compromised — revoke it immediately, no waiting.

    New to SSH? Start at the top. Already have keys set up? Jump to [Testing Your Connection](#testing-your-connection)

---

SSH authentication is one of those things most developers set up once by copying a command they found online and never fully understand — until something breaks, or until they realize they have been doing it wrong the whole time. This section explains what SSH keys actually are, why they matter for your security, and how to set them up correctly from scratch — on any machine, any operating system.

---

## What Is an SSH Key and Why Does It Matter?

SSH stands for Secure Shell. It is a protocol for creating encrypted, authenticated connections between two machines — in this case, between your computer and GitHub. When you use SSH, you are telling GitHub: "I am who I claim to be, and here is the cryptographic proof."

!!! tip "Never used a terminal before?"
    A terminal is a text-based interface where you type commands directly to your computer. Every operating system has one.

    - **macOS:** Press `Cmd + Space`, type `Terminal`, hit Enter.
    - **Linux:** Look for Terminal in your applications, or press `Ctrl + Alt + T`.
    - **Windows:** Search for `Git Bash` (install Git for Windows first if you haven't) or use `PowerShell`.

    When you open it you will see a blinking cursor waiting for input. That is normal. You type a command and press Enter to run it.

The way it works is sophisticated. When you generate an SSH key, you actually generate two things at once — a ***private key*** and a ***public key***. They are mathematically linked, like two sides of the same lock.

- The ***public key*** can be shared freely. You give it to GitHub. Think of it as the lock on the door.
- The ***private key*** never leaves your machine. Think of it as the only key that opens that lock. ***Never share it. Never copy it anywhere. Never upload it to anything.***

When you try to connect to GitHub, GitHub sends your machine a cryptographic challenge — a unique mathematical puzzle generated just for that moment. Your private key solves that puzzle in a way that only the matching private key could. GitHub then verifies the solution using your public key. If it checks out, you are authenticated. No password travels over the network. Nothing can be intercepted.

!!! tip "How is this different from a CAPTCHA?"
    You have probably solved CAPTCHAs before — those "click all the traffic lights" puzzles. The concept of a challenge-and-response is similar, but there are two critical differences.

    First, a CAPTCHA proves you are human — it does not prove you are *you*. Anyone sitting at a computer can solve one. An SSH challenge proves cryptographic ownership of a specific private key that exists on one specific machine.

    Second, a CAPTCHA can be faked or bypassed with enough effort. The math behind SSH cannot. The private key is the only thing in existence that can produce the correct response to GitHub's challenge. There is no guessing, no brute forcing at any practical scale, no workaround.

This is why SSH is significantly stronger than using a password or a personal access token for your daily Git work. A stolen password works from anywhere in the world. A stolen SSH key requires physical or remote access to the machine where the private key lives — and if you protect it with a passphrase (which you will), even that is not enough.

!!! warning "The most common SSH mistake"
    Generating a key ***without a passphrase***. It feels more convenient because you never have to type anything — but it means anyone who gets access to your machine, even briefly, has full access to every service your SSH key is authorized for. Your SSH agent handles the typing for you so you get the security without the friction. ***Always use a passphrase.***

---

## Choosing the Right Key Type

There are several SSH key algorithms available. The one you want is ***ed25519***. Here is why.

**RSA** is the oldest and most widely supported. For years it was the default recommendation. It still works, but requires large key sizes (4096 bits minimum to be considered secure today) and is slower than modern alternatives.

**ed25519** is based on elliptic curve cryptography. It produces much shorter keys that are faster to generate, faster to verify, and considered more secure than equivalent RSA keys. A 256-bit ed25519 key is stronger than a 3072-bit RSA key. It is the current best practice and is supported by GitHub, GitLab, Bitbucket, and every major SSH implementation.

**ECDSA** is another elliptic curve option but has theoretical concerns around its random number generation that ed25519 does not share. Skip it.

***Use ed25519. If you have old RSA keys, they are not broken — but generate ed25519 for anything new.***

---

## Generating Your SSH Key

Open your terminal and run this command. It works the same on macOS, Linux, and Windows Git Bash.

```
ssh-keygen -t ed25519 -C "your-noreply@users.noreply.github.com"
```

Breaking this down so it makes sense:

- `ssh-keygen` — the tool that generates SSH keys
- `-t ed25519` — tells it to use the ed25519 algorithm
- `-C "..."` — adds a comment so you can identify the key later

Use your GitHub noreply address for the comment — not your real email. This comment ends up inside your public key file and could be visible in certain contexts.

!!! tip "Where do I find my GitHub noreply address?"
    Go to GitHub → Settings → Emails → look for the address that ends in `@users.noreply.github.com`. It looks something like `123456789+yourusername@users.noreply.github.com`. Copy that and use it in the command above.

**When prompted where to save the key:**

The default location is fine for most people. Just press Enter to accept it.

```
macOS default:           /Users/youruser/.ssh/id_ed25519
Linux default:           /home/youruser/.ssh/id_ed25519
Windows default:         C:\Users\youruser\.ssh\id_ed25519
```

!!! tip "What does ~/.ssh/ mean?"
    The `~` symbol is shorthand for your home directory — the folder that belongs to your user account on the computer. So `~/.ssh/` means "the .ssh folder inside your home directory." You do not need to create this folder manually — ssh-keygen creates it for you if it does not exist.

If you have multiple GitHub accounts or use SSH for multiple services, give each key a descriptive name instead of the default:

```
/home/youruser/.ssh/id_ed25519_github_personal
/home/youruser/.ssh/id_ed25519_github_work
```

**When prompted for a passphrase:**

***Use one. Make it strong.*** Here is how to think about passphrase strength:

- **Minimum baseline** — 16 characters. Mixed uppercase, lowercase, numbers, and at least one symbol. This is the floor, not the goal.
- **Solid protection** — 20 to 32 characters. Randomized, no dictionary words on their own, alternating character types throughout.
- **High-value or high-risk accounts** — go up to 64 characters. If your GitHub account has a significant public presence, contains proprietary code, or connects to production systems, treat your SSH passphrase like the master key it is.

A strong strategy that is actually memorable: pick three or four completely unrelated words, add numbers and symbols between them, and capitalize unpredictably. Something like `Riv3t!Candle$Moon42` — not that exact phrase, but that structure. The randomness between the words is what makes it strong. The words themselves are what make it memorable.

Store it in your password manager. Do not rely on memory alone for a 32+ character passphrase.

Your SSH agent will handle entering it automatically after the first time, so you will not be typing it constantly — but it is there protecting you when it matters most.

!!! warning "Using AI to help with your SSH setup — what to share and what to never share"
    AI tools can be genuinely useful for troubleshooting SSH issues — explaining error messages, walking through configuration, and diagnosing problems. But there are things you must ***never*** share with any AI, ever, under any circumstances.

    ***Never share with any AI:***
    - The contents of your private key file (the one without .pub)
    - Your SSH passphrase
    - Your actual key fingerprint in a context where it could identify you
    - The full contents of ~/.ssh/config if it contains real account usernames or sensitive hostnames

    ***Safe to share with AI for troubleshooting:***
    - Error messages (with personal details removed)
    - The structure of your config file with usernames replaced by placeholders
    - The output of `ssh -T git@github.com` with your username replaced
    - General questions about how SSH works

    The AI does not need to see your actual secrets to help you solve a problem. Describe the issue, share the error, replace real values with placeholders. If an AI ever asks you to paste your private key or your passphrase for any reason — that is a red flag. Stop immediately.

    This applies to every AI tool — Claude included.

When the command finishes, you will have two files:

```
id_ed25519      ← your PRIVATE key.
                  Never share this. Never copy this anywhere.
                  Never upload it. Never paste it into anything.

id_ed25519.pub  ← your PUBLIC key.
                  This is the one you give to GitHub.
                  The .pub extension means public.
```

!!! warning "This is the most important distinction in this entire section"
    ***The file WITHOUT .pub is your private key. It never leaves your machine.***
    ***The file WITH .pub is your public key. This is what goes to GitHub.***

    If you ever find yourself copying the contents of a file that does NOT end in `.pub` and pasting it somewhere — stop. That is the wrong file. The damage from sharing a private key cannot be undone by deleting it from wherever you pasted it. Revocation and replacement is the only fix.

---

## Adding Your Key to the SSH Agent

The SSH agent is a small background process that holds your decrypted private key in memory. Without it, you would need to type your passphrase every single time you push, pull, or clone. With it, you enter your passphrase once and the agent handles everything else for the rest of your session.

Think of it like a secure keychain app on your phone — you unlock it once with your fingerprint, and it handles all the passwords inside without making you re-enter them every time.

**macOS:**

macOS has a built-in SSH agent that integrates with Keychain, meaning your passphrase persists even across reboots.

```
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

You should also create or edit `~/.ssh/config` and add:

```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

!!! tip "How do I create or edit ~/.ssh/config?"
    In your terminal, type:
    ```
    nano ~/.ssh/config
    ```
    This opens a simple text editor. Paste the configuration above, then press `Ctrl+O` to save and `Ctrl+X` to exit. If the file already exists, add the lines to what is already there — do not replace existing content without reading it first.

**Linux:**

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

On Linux the agent does not persist across reboots by default. Add `ssh-add ~/.ssh/id_ed25519` to your shell profile (`~/.bashrc` or `~/.zshrc`) so it loads automatically, or use a keyring manager like GNOME Keyring.

**Windows (Git Bash):**

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

To make the agent start automatically on Windows, open PowerShell as Administrator and run:

```
Set-Service -Name ssh-agent -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

After running any of the above, you should see:

```
Identity added: /home/youruser/.ssh/id_ed25519
```

That confirms your key is loaded and the agent is running.

---

## Adding Your Public Key to GitHub

Now you need to give GitHub your ***public key*** — the `.pub` file — so it knows to trust connections from your machine.

First, display the contents of your public key:

```
macOS / Linux:
cat ~/.ssh/id_ed25519.pub

Windows (Git Bash):
cat ~/.ssh/id_ed25519.pub

Windows (PowerShell):
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

You should see a single line of text that starts with `ssh-ed25519` and ends with your comment:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... 123456789+yourusername@users.noreply.github.com
```

***Select and copy the entire line — from ssh-ed25519 all the way to the end.***

!!! warning "Double-check what you are copying"
    The output of `cat ~/.ssh/id_ed25519.pub` should be a single line starting with `ssh-ed25519`. If you accidentally run `cat ~/.ssh/id_ed25519` without the `.pub` and see multiple lines starting with `-----BEGIN OPENSSH PRIVATE KEY-----` — ***stop immediately***. That is your private key. Do not copy it anywhere. Run the command again with `.pub` at the end.

Now add it to GitHub:

```
GitHub → Settings → SSH and GPG keys → New SSH key

Title:     Something descriptive that tells you which machine this is.
           Examples: "Personal MacBook 2024", "Work Laptop", "Home Desktop"

Key type:  Authentication Key

Key:       Paste your public key here — the full line starting with ssh-ed25519
```

Click Add SSH key. GitHub will ask for your password to confirm.

!!! tip "Name your keys clearly"
    When you audit your SSH keys later — which you should do every few months — you need to be able to identify every single key at a glance. A key named "my key" or "laptop" becomes useless when you have had three laptops over the years. Name it with the machine, the year, and the purpose if needed. "Personal MacBook 2024 - GitHub" is never confusing.

---

## Testing Your Connection

Before you try to push anything, confirm the connection actually works:

```
ssh -T git@github.com
```

The first time you connect, you may see:

```
The authenticity of host 'github.com (140.82.121.4)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Type `yes` and press Enter. This adds GitHub to your list of known hosts so you are not asked again. This is normal on first connection to any new server.

If everything is working you will see:

```
Hi yourusername! You've successfully authenticated, but GitHub does not provide shell access.
```

The "does not provide shell access" part is normal. GitHub uses SSH for Git operations only. The important part is "successfully authenticated."

!!! warning "If you see 'Permission denied (publickey)'"
    This means GitHub does not recognize your key. Work through this checklist:

    - Did you copy the full public key including the `ssh-ed25519` prefix and the comment at the end?
    - Did you save it under the correct GitHub account?
    - Is your SSH agent running? Check with: `ssh-add -l`
    - Is the key loaded? You should see your key listed in the output.
    - If you see "The agent has no identities" — run `ssh-add ~/.ssh/id_ed25519` again.

    If you have multiple keys and a custom SSH config, test with your alias:
    ```
    ssh -T git@github-youraliasname
    ```

---

## Configuring ~/.ssh/config for Multiple Accounts

If you have more than one GitHub account — a personal account and a work account, a public portfolio and a private projects account — you need to tell SSH which key to use for which account. Without this, SSH always tries the default key and you will get authentication errors or accidentally push under the wrong identity.

This is the step most SSH tutorials skip entirely. It is also the source of most "why isn't this working" questions when people have multiple accounts.

Edit or create `~/.ssh/config`:

```
macOS / Linux:
nano ~/.ssh/config

Windows (PowerShell):
notepad $env:USERPROFILE\.ssh\config
```

Add an entry for each GitHub account:

```
# Personal GitHub account
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  AddKeysToAgent yes

# Work GitHub account
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
  AddKeysToAgent yes
```

The `Host` value is an alias you choose. It replaces `github.com` in your Git commands:

```
# Clone using personal account:
git clone git@github-personal:yourusername/your-repo.git

# Clone using work account:
git clone git@github-work:yourworkusername/work-repo.git
```

For repos you have already cloned, update the remote URL:

```
git remote set-url origin git@github-personal:yourusername/your-repo.git
```

Test each account separately:

```
ssh -T git@github-personal
ssh -T git@github-work
```

Each should respond with the correct username for that account.

!!! warning "Git may still use the wrong identity"
    Even with SSH configured correctly, git itself may still attach the wrong name and email to commits if your global git config points to one identity. For repos that should use a different account, set the identity at the repo level:

    ```
    cd your-repo
    git config user.name "Your Work Name"
    git config user.email "worknoreply@users.noreply.github.com"
    ```

    This overrides the global config for that repo only and does not affect your other repos.

---

## Setting Up SSH Commit Signing

GitHub lets you use your SSH key not just for authentication but also for ***signing commits***. This is what produces the green "Verified" badge — cryptographic proof that a commit came from you specifically and was not altered after the fact.

We covered enabling Vigilant Mode in [Securing Your GitHub Account](account.md). Vigilant Mode makes unsigned commits show as unverified. SSH signing makes your commits show as verified. Together they mean anyone reviewing your repo can immediately see which commits are genuinely yours.

***This is not just cosmetic.*** Without signing, anyone with push access to a shared repo could make a commit that looks identical to yours. Signing makes impersonation immediately visible.

**Step 1 — Tell git to use SSH for signing:**

```
git config --global gpg.format ssh
```

**Step 2 — Tell git which key to sign with:**

```
macOS / Linux:
git config --global user.signingkey ~/.ssh/id_ed25519.pub

Windows:
git config --global user.signingkey "$env:USERPROFILE\.ssh\id_ed25519.pub"
```

**Step 3 — Enable automatic signing on every commit:**

```
git config --global commit.gpgsign true
```

With this set, every commit you make is automatically signed. No extra flags needed.

**Step 4 — Add your signing key to GitHub:**

```
GitHub → Settings → SSH and GPG keys → New SSH key

Title:     Same machine name as your auth key, add "signing"
           Example: "Personal MacBook 2024 - signing"

Key type:  Signing Key

Key:       Paste the same public key — full line starting with ssh-ed25519
```

You can use the same key for both authentication and signing.

**Step 5 — Verify it is working:**

Make a test commit and push it. Go to your repo on GitHub, click the commit hash, and look for a green "Verified" badge. If you see it — you are done.

!!! tip "Seeing 'Unverified' even after setup?"
    Make sure you added the key as a ***Signing Key*** in GitHub settings — not just as an Authentication Key. They are separate entries and both are needed. Also confirm `git config --global commit.gpgsign` returns `true`.

---

## If Your SSH Key Is Compromised

If you have any reason to believe your private key has been exposed — your machine was stolen, you found a copy of it somewhere unexpected, someone had unsupervised access to your machine — ***treat it as compromised immediately***.

Do not wait to see if anything suspicious happens. By the time you notice suspicious activity it is already too late.

```
Step 1 — Revoke the key on GitHub immediately:
GitHub → Settings → SSH and GPG keys → Delete

Step 2 — Generate a new key:
Follow this section from the beginning.
Use a new passphrase — not the same one.

Step 3 — Add the new key to GitHub and every other
service that used the old key.

Step 4 — Audit your GitHub security log:
GitHub → Settings → Security log
Look for pushes, repo changes, or access events
you do not recognize.

Step 5 — If anything looks suspicious:
Change your GitHub password.
Revoke all active sessions:
GitHub → Settings → Sessions → Revoke all
Rotate your 2FA if needed.
```

!!! warning "Deleting the key from your machine is not enough"
    If you delete `~/.ssh/id_ed25519` from your computer but do not remove the public key from GitHub, the key is still trusted by GitHub. Anyone who has a copy of your private key can still authenticate. ***Revocation happens on GitHub — not on your machine.*** Always do both.

---

## Quick Reference

```
Generate a new ed25519 key:
ssh-keygen -t ed25519 -C "noreply@users.noreply.github.com"

Add to agent — macOS:
eval "$(ssh-agent -s)"
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

Add to agent — Linux / Windows Git Bash:
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

View your public key to copy to GitHub:
cat ~/.ssh/id_ed25519.pub

Test your connection:
ssh -T git@github.com

Check which keys are loaded in the agent:
ssh-add -l

Configure commit signing:
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

---

*[@sudochef](https://instagram.com/sudochef) — Build like you're the target. Because you are.*
