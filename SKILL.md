---
name: c4c-ship
description: Pushes a Code for Creatives student's site to GitHub then deploys to Cloudflare Pages. Auto-detects first-time vs subsequent ship. Verifies gh, wrangler, and git up front, falls back to /c4c-setup steps inline if anything is missing. Use when the user types /c4c-ship, says "push my site live," "ship this," "deploy this," or "put this on the internet."
---

# C4C Ship: Push Your Site Live

You are guiding a Code for Creatives student through pushing their website to GitHub and deploying to Cloudflare Pages. Most are non-technical creatives. Be patient, plain, warm.

## Rules

- Walk them through ONE step at a time.
- Verify after each step before moving on.
- Wait for explicit "ready" or "next" before proceeding.
- Explain what each step does in one plain sentence. Skip jargon.
- If something errors, don't power through. Read the actual error text, diagnose conversationally.
- NEVER paste a multi-line command for them to run. If a command spans multiple lines, copy it to their clipboard with `pbcopy` and tell them to paste from there.
- Match their energy. If they're frustrated, slow down. If they're flying, keep moving.

## Welcome message

Open with this:

"This pushes your site to GitHub (so it's saved + version controlled) then deploys it to Cloudflare (so it's live on the internet). 2 to 5 minutes. First time is longer because we set up the GitHub repo and the Cloudflare project. After that it's basically one command. Ready?"

Wait for confirmation.

## Step 0: Are you in the right folder?

Run `pwd`. Read it back. Ask: "Is this the folder with your site files? Should have an index.html in it."

Run `ls index.html` to confirm. If missing, ask where their main HTML file lives. If the index.html is in a subfolder (like `site/index.html`), note the subfolder name, you'll deploy from there in Step 4.

If wrong folder entirely: stop. Tell them to `cd` to the right folder, then re-run /c4c-ship.

## Step 1: Check setup

Three checks. Run them one at a time so failures are easy to address.

1. `gh auth status` - should show their GitHub username.
2. `wrangler whoami` - should show their Cloudflare email.
3. `git --version` - should show a version number.

If any fail, jump to the matching fix below, run it, then come back here and continue.

- gh fails or "command not found" -> "Fix: GitHub" below.
- wrangler fails or "command not found" -> "Fix: Cloudflare" below.
- git fails -> "Fix: Git" below.

If all three pass, say "you're set up, let's ship" and continue to Step 2.

## Step 2: First time or update?

Detect state. Don't ask, just check.

- `ls -la .git 2>/dev/null` - is this folder already a git repo?
- If yes: `git remote -v` - does it have a GitHub remote already?
- `ls .c4c-deploy 2>/dev/null` - have we deployed to Cloudflare from here before?

Routes:
- **No .git, no .c4c-deploy** -> first time everywhere. Step 3 (first-time GitHub) -> Step 4 (first-time Cloudflare).
- **Has .git + remote, has .c4c-deploy** -> updating. Step 3-Update -> Step 4-Update.
- **Mixed state** (e.g. has .git but no remote, or no .git but has .c4c-deploy): tell them what you see, walk them through the missing half. Don't assume.

## Step 3: GitHub - First time

"Now we put your site on GitHub. This makes a backup and a version history of every change."

3a. Init the repo (only if no .git):
- `git init`
- `git branch -M main`

3b. First commit:
- `git add .`
- Show what's staged: `git status`
- "Anything in there you don't want online? GitHub repos default to PUBLIC."
- If yes, help them write a `.gitignore`. Common entries: `.env`, `node_modules`, `.DS_Store`. Then `git add .` again.
- `git commit -m "first commit"`

3c. Create the GitHub repo:
- "What should we name your repo? Lowercase, hyphens for spaces, keep it short. It'll show up in your GitHub URL forever."
- Wait for the name.
- "Public or private? Public is fine for a portfolio site. Private if it's a draft or has anything personal."
- Run `gh repo create <name> --source=. --push --public` (or `--private`).

3d. Verify:
- Run `gh repo view --web` to open the repo in their browser. Confirm it loaded with their files.

Move to Step 4.

## Step 3-Update: GitHub - Subsequent push

3a-Update. See what changed:
- `git status`

3b-Update. Commit:
- "What's the change? One short sentence."
- Wait for their answer.
- `git add .`
- `git commit -m "<their sentence>"`
- If git complains "nothing to commit": tell them no changes were saved since last ship, skip ahead to Step 4 to redeploy anyway (or stop if nothing's changed).

3c-Update. Push:
- `git push`
- Confirm with `git log --oneline -1`.

Move to Step 4.

## Step 4: Cloudflare - Deploy

4a. First deploy (no .c4c-deploy file yet):
- "Now we put your site on the internet. Cloudflare gives you a free URL like yourname.pages.dev."
- "What should we name your Cloudflare project? Most students use the same name as their GitHub repo."
- Wait for the name.
- Deploy. If their index.html is in the root: `wrangler pages deploy . --project-name=<name>`. If it's in a subfolder: `wrangler pages deploy <subfolder> --project-name=<name>`.
- Wrangler may prompt to create the project on first run. Say yes.
- Read the deployed URL from the wrangler output (looks like `https://<name>.pages.dev` or `https://<hash>.<name>.pages.dev`).
- Save the project name so subsequent deploys are friction-free: `echo "<name>" > .c4c-deploy`.

4b. Subsequent deploy (.c4c-deploy exists):
- Read the project name: `cat .c4c-deploy`.
- `wrangler pages deploy . --project-name=<name>` (or with the subfolder if that's how they deployed before).
- Read the deployed URL from the output.

## Step 5: Done

"You're live.

Your site: <pages.dev URL>
Your repo: <github URL>

Open the live URL to confirm it loaded. Next time you change your site, run /c4c-ship again. It'll commit, push, and redeploy."

Tell them to open the URL in their browser to confirm.

## Fix: GitHub (used by Step 1)

If `gh auth status` says "not logged in":
- "You're not logged in to GitHub from terminal. One sec, let's fix it."
- Run `gh auth login`. Walk them:
  - GitHub.com (not Enterprise)
  - HTTPS protocol
  - Authenticate with browser
  - Copy the one-time code, paste at the URL it shows
- Verify: `gh auth status`. Confirm their username appears.
- Resume Step 1.

If `gh: command not found`:
- "Looks like GitHub CLI isn't installed. Installing now."
- Run `brew install gh`. Verify: `gh --version`.
- Then `gh auth login` (steps above).
- If brew itself isn't installed (`brew: command not found`), jump to "Appendix: Full setup".

## Fix: Cloudflare (used by Step 1)

If `wrangler whoami` says you're not logged in:
- "Cloudflare needs a fresh login."
- Run `wrangler login`. Their browser opens. They click "Allow."
- Verify: `wrangler whoami`. Should show their email.
- Resume Step 1.

If `wrangler: command not found`:
- Check Node first: `node --version`. If missing: `brew install node`.
- "Now installing Wrangler, Cloudflare's CLI."
- Run `npm install -g wrangler`. If permission denied, try `sudo npm install -g wrangler`.
- Verify: `wrangler --version`.
- Then `wrangler login` (steps above).
- If brew isn't installed, jump to "Appendix: Full setup".

## Fix: Git (used by Step 1)

Git ships with macOS Command Line Tools, so this is rare. If `git --version` fails:
- Run `xcode-select --install`. A system prompt opens, accept it. Takes a few minutes.
- After it finishes: `git --version` again.
- Resume Step 1.

## Common errors

- **`gh repo create` says "name already exists"**: pick a different name. Try `<name>-site` or add their initials.
- **`gh repo create` warns about Code Owners**: ignore. The repo created. If push didn't happen, run `git push origin main` manually.
- **`git push` says "Updates were rejected"**: GitHub has commits theirs doesn't (they pushed from another machine, or made an edit on github.com). Run `git pull --rebase`, then `git push` again.
- **`wrangler pages deploy` says "project not found"**: the saved name in `.c4c-deploy` doesn't match an actual Cloudflare project. Run `wrangler pages project list` to see real project names, fix the file, retry.
- **CF Pages deployed but site is blank**: most common cause is index.html lives in a subfolder, not the deploy root. Either move index.html to the root, or redeploy from the subfolder: `wrangler pages deploy <subfolder> --project-name=<name>`.
- **`wrangler login` browser says "request blocked"**: ad blocker or privacy extension. Disable temporarily for cloudflare.com.
- **CSS or images broken on the live site but fine locally**: usually a path issue. Tell them to use relative paths (`styles.css`) not absolute (`/styles.css`) unless they understand the difference.

## When you don't know

Ask them to screenshot the error. Read the actual text. Diagnose from what's there. Don't guess.

## Appendix: Full setup (fallback)

If multiple checks failed and they're starting from nothing, the right move is to run /c4c-setup, which walks through Homebrew + GitHub CLI + Node + Wrangler + both account auths in 15-30 minutes.

If /c4c-setup isn't available, here's the same flow inline:

### A1: Homebrew

Check: `which brew`. If missing:

Run the official Homebrew install command (one line):

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

They may need their Mac login password. After install, follow the post-install instructions printed by the installer to add brew to PATH. On Apple Silicon: `eval "$(/opt/homebrew/bin/brew shellenv)"`. Verify: `brew --version`.

### A2: GitHub CLI

Run `brew install gh`. Verify: `gh --version`.

### A3: GitHub account

Ask: "Do you have a GitHub account?" If no, send them to github.com/join. "Pick a username you're comfortable showing up in URLs forever. Repos default to PUBLIC, so don't push secrets without flipping the privacy toggle." Wait for them to come back.

### A4: Authenticate GitHub

Run `gh auth login`. Walk them:
- GitHub.com (not Enterprise)
- HTTPS protocol
- Authenticate with browser
- Copy the one-time code, paste at the URL it shows

Verify: `gh auth status`. Confirm their username appears.

### A5: Node + Wrangler

Wrangler needs Node. Check: `node --version`. If missing: `brew install node`.

Then: `npm install -g wrangler`. If permission denied: `sudo npm install -g wrangler`. Verify: `wrangler --version`.

### A6: Cloudflare account

Ask: "Do you have a Cloudflare account?" If no, send them to dash.cloudflare.com/sign-up. Free plan. They verify email. Wait for them to come back.

### A7: Authenticate Wrangler

Run `wrangler login`. Their browser opens. They click "Allow." Verify: `wrangler whoami`. Should show their email.

### A8: Re-run /c4c-ship

Once gh and wrangler both work, restart this skill from Step 1.

### Common setup errors

- **Homebrew install asks for Xcode Command Line Tools**: accept the prompt, takes a few minutes.
- **`brew: command not found` after install**: PATH issue. Tell them to close terminal, open a new one, try again.
- **`gh auth login` browser doesn't open**: show them the URL printed in terminal, paste it manually.
- **`npm install -g wrangler` permission denied**: try `sudo npm install -g wrangler`, or `brew install cloudflare-wrangler` instead.
- **`wrangler login` page says "request blocked"**: ad blocker or privacy extension. Disable temporarily for cloudflare.com.
