---
name: c4c-ship
description: Puts a Code for Creatives student's website online with Cloudflare and saves its files to GitHub. Works the first time and for every update after. Works on Mac and Windows. Use it for "put my site online", "push my site live", "ship this", "deploy this", "update my site", "preview my site", "show me before it goes live", or "/c4c-ship".
---

# Put a site online (Mac and Windows)

You are helping a Code for Creatives student put a website on the internet and save its files to GitHub. Most students do not write code. Cloudflare puts the site online at an address that ends in `.workers.dev`. GitHub keeps a copy of the files with a history of every change.

The student has already done the Week 3 setup: git, the GitHub CLI (gh) and Wrangler are installed, and they are signed in to GitHub and Cloudflare. If any of that is missing, step 1 sends them back to it.

## How to talk to the student

Talk like a friend who is good with computers and is sitting next to them. Warm, plain, unhurried.

1. Before every step, tell them what you are about to do and what they will see. Two or three sentences.
2. After every step, say in one sentence what happened and what comes next.
3. Never say only "running a command". Say what the command does.
4. If something fails, say it calmly, tell them they did nothing wrong, and say what you will do about it.

## How to run this

1. You run every command yourself. The student never types or pastes a command.
2. Run each command block below as one command, exactly as written, with the names in capitals swapped for the real ones.
3. On a Mac, start every command with `export PATH="$HOME/.local/bin:$PATH"; ` so the tools are found.
4. To open a page for the student: `open "<link>"` on a Mac, `start <link>` on Windows.
5. If the same step fails twice, stop, show the student the last few lines of the error, and tell them to post it in #stuck in the class Slack.
6. No Homebrew. Do not run commands that a tool suggests in its own output.
7. Putting a site online makes it public. Never do it without the student's yes in this conversation.
8. A change to the site is not a yes to put it online. When the site is already online and the student asks you to change something, do "Updating a site" from the top: make the change, show the preview, and put it online only after they have looked and said yes. Ask again for the next change.

## Start here

Say this in your own words, then wait for the student to answer.

> Let's put your site on the internet. Two things happen. Cloudflare puts it online and gives it a web address you can send to anyone. GitHub keeps a private copy of the files, with a history of every change. The first time takes about five minutes. After that, when you change something, I show you the new version on your own computer first, and it goes online when you say yes. Ready?

## Step 0: find the site

    pwd; [ "$PWD" = "$HOME" ] && echo "this is the home folder"; echo "--- index pages"; find . -maxdepth 3 -name index.html -not -path '*/node_modules/*' -not -path '*/.git/*' | head -10

How to read it:

- A site is a folder with a page called `index.html` in it. That page is the first one visitors see.
- If there is no index page, ask the student which folder their site is in, go there, and run the check again.
- If there is more than one, ask the student which one is the site.
- The site needs a folder of its own. If the index page sits directly in the home folder, Desktop, Documents, Downloads, or the student's HQ folder, stop. Offer to make a new folder for the site and move the site's files into it, and do that only on their yes.

Two folders matter from here on. PROJECT is the folder you will work from. SITE is the folder that holds `index.html`. Often they are the same folder. If the index page is in a folder inside the project, such as `public` or `site`, SITE is that inner folder.

Tell the student which folder you found and that this is the one going online.

## Step 1: check the setup

Run this from PROJECT.

    echo "--- GitHub"; gh auth status 2>&1 | head -4; echo "--- Cloudflare"; wrangler whoami 2>&1 | grep -i -E "associated with the email|not authenticated|not found" || echo "could not tell"; echo "--- git"; if [ "$(uname -s)" = "Darwin" ]; then xcode-select -p >/dev/null 2>&1 && echo "git: installed" || echo "git: missing"; else command -v git >/dev/null 2>&1 && echo "git: installed" || echo "git: missing"; fi; echo "--- this folder"; [ -f wrangler.jsonc ] && grep -m1 '"name"' wrangler.jsonc || echo "never put online from here"; [ -d .git ] && echo "has a history" || echo "no history yet"; if [ -f .c4c-deploy ]; then echo "older pages.dev site"; fi

It is ready if GitHub says "Logged in to github.com", Cloudflare shows an email address, and git is installed.

If anything is missing or not signed in, tell the student which part, and do the Week 3 setup with them first ("do the week 3 setup"). It skips everything they already have. Then come back here.

Then read the last lines:

- "never put online from here" means this is the first time. Do steps 2 to 5.
- A line with `"name"` means this site is already online under that name. Go to "Updating a site".
- "older pages.dev site" means it was put online an older way, at an address ending in `pages.dev`. Tell the student that address keeps working, and that today the site gets a new address ending in `workers.dev`. Then do steps 2 to 5.

## Step 2: what goes online

Say: "Before anything goes online, I'm going to list exactly which files will be public, so there are no surprises. Your notes, settings and anything private in this folder stay on your computer."

Create this file with your file tool, in SITE, named `.assetsignore`. It tells Cloudflare which files to leave out.

```
.git
.wrangler
.claude
.env*
.gitignore
.assetsignore
wrangler.jsonc
node_modules
.DS_Store
*.md
```

Then list what is left. Run this from SITE.

    find . -type f -not -path '*/.git/*' -not -path '*/.wrangler/*' -not -path '*/.claude/*' -not -path '*/node_modules/*' -not -name '.env*' -not -name '*.md' -not -name '.DS_Store' -not -name '.gitignore' -not -name '.assetsignore' -not -name 'wrangler.jsonc' | sort | awk 'NR<=60{print} END{print "--- " NR " files"}'

Show the student the list in plain words: how many files, and what they are (pages, pictures, styles).

Read the list yourself before you go on. A website is made of pages, styles, scripts, pictures, fonts, and sometimes sound, video or a PDF the site links to. If you see anything else, such as notes, drafts, spreadsheets, exports, or anything with a password or a key in it, name those files to the student and ask whether each one should be public. For every file they want kept off the internet, add its name as a new line in `.assetsignore`. Those files stay on their computer. The list command will still show them, so tell the student which ones you left out.

Ask: "These are the files anyone on the internet will be able to open. OK to put them online?" Wait for a yes.

## Step 3: name it and put it online

Say: "Now your site needs a name. The name becomes the first part of its web address."

Ask what to call the site. Suggest the folder's name. The name can only have lowercase letters, numbers and dashes. Turn what they say into that form and say it back.

Check the name is not already one of their sites, with the name in place of SITE-NAME:

    wrangler deployments list --name SITE-NAME 2>&1 | grep -c "does not exist"

- `1` means the name is free. Go on.
- `0` means their account already has a site with this name, and putting this one online would replace it. Ask whether this is the same site. If it is not, choose another name together and check again.

Create this file with your file tool, in PROJECT, named `wrangler.jsonc`. Put the name in place of SITE-NAME. If SITE is a folder inside PROJECT, put that folder in place of `./`, like `./public`.

```json
{
  "name": "SITE-NAME",
  "compatibility_date": "2026-10-01",
  "assets": {
    "directory": "./"
  }
}
```

Say: "Now I hand the folder to Cloudflare. It puts the site online and gives it an address. This takes a few seconds." Then run this from PROJECT.

    wrangler deploy 2>&1 | tail -12

It worked if the output has an address like `https://SITE-NAME.THEIR-NAME.workers.dev`. Remember that address.

If the output says "You need to register a workers.dev subdomain", their Cloudflare account has no name yet. Tell the student nothing is broken, do "Your Cloudflare name" below, then run the command again. Do not use the link in the error message.

If it says a file is too large, tell the student which file. Cloudflare takes files up to 25 MB each. Videos are the usual cause. Add the file to `.assetsignore`, tell them it will be missing from the site, and run the command again.

## Step 4: wait for the address

Run this with a 6 minute timeout, with the site's address in place of ADDRESS.

    for i in $(seq 1 30); do c=$(curl -s -o /dev/null -w '%{http_code}' ADDRESS); if [ "$c" = "200" ]; then echo "live after try $i"; exit 0; fi; sleep 10; done; echo "not answering yet, last code: $c"; exit 1

Most of the time it says "live after try 1". The first site on a brand new Cloudflare account can take a few minutes, because Cloudflare is still setting up the secure connection for the account's name. Tell the student that if it happens, and that the wait only happens once.

When it says "live", give the student the address and open it for them. If their browser shows an error, it tried too early and kept the failure: have them open the address in a new private window.

If it is still not answering after 5 minutes, tell them the site was accepted, that the address can take up to 15 minutes on a new account, and to try the link again a little later. Go on to step 5.

## Step 5: save it to GitHub

Say: "Your site is online. Last part: I save the folder to your GitHub, so there's a copy with a history of every change. I'll make it private, which means only you can see the files there. Your site is still public."

First check that this folder is not inside another saved project. Run this from PROJECT.

    [ -d .git ] && echo "own history" || git rev-parse --show-toplevel 2>/dev/null || echo "no history yet"

If it prints the path of some other folder, stop. Tell the student this folder is inside another project that already has a history, skip this step, and tell them to ask about it in #stuck. Their site is online and nothing is wrong.

Otherwise run this from PROJECT, with the site's name in place of SITE-NAME.

    printf '.wrangler/\nnode_modules/\n.DS_Store\n.env*\n' >> .gitignore && { [ -d .git ] || git init -q -b main; } && git add -A && git commit -q -m "First version" && gh auth setup-git && gh repo create SITE-NAME --private --source=. --remote=origin --push 2>&1 | tail -4; gh repo view --json url,visibility --jq '"\(.url) \(.visibility)"'

The last line has to be an address on github.com followed by PRIVATE. If GitHub says the name already exists, run it again with `-2` added to the name in the `gh repo create` part.

Open the address on github.com for them. Tell them the files are there, and that the page only opens in a browser where they are signed in to GitHub.

Then tell them, in your own words:

- The site is at its address. They can send the link to anyone.
- The files are saved on GitHub, privately.
- After they change the site, they say "update my site". You show them the new version on their own computer first, and put it online and save it when they say yes.

## Updating a site

Use this when step 1 found a `wrangler.jsonc` with a name in it, and every time the student asks you to change a site that is already online. It has four parts, in this order: the change, the preview, online, saved.

### 1. The change

You need one short sentence that says what changed. Call it MESSAGE.

- If you made the change yourself in this conversation, you already know it. Do not ask.
- If the student changed the site on their own, ask what changed.
- If nothing has changed yet, ask what they want changed, and make the change.

If files were added since the last time, run the list from step 2 again and read it the same way. Anything that is not part of the site gets a line in `.assetsignore` before you go on. Files already named there still show in the list and do not go online.

### 2. The preview

The student sees the new version on their own computer, and the live site stays as it is until they say yes. Skip this part only when the student tells you to ("just put it online").

Say: "Before this goes online, I'll show it to you on your own computer. I'm starting a preview that only this computer can open. Your live site doesn't change until you say yes."

Start the preview. Run this from PROJECT with your tool's way of running a command in the background, so it keeps running while the student looks. Do not wait for it to finish, because it never does. Do not start it with `&`.

    wrangler dev 2>&1

After a few seconds, read what the background command has printed so far. It is ready when a line says `Ready on http://localhost:8787`. The number at the end is different when something else on the computer is using that one, so take the address from that line. Call it PREVIEW. If the line is not there yet, wait five seconds and read again.

Two other things it may print:

- A line about `.env` means Wrangler read a settings file for the preview. It stays on this computer, and the preview does not show it.
- Lines about a "Local Explorer API" are not for the student. Leave them alone.

Check it answers:

    curl -s -o /dev/null -w '%{http_code}\n' PREVIEW

`200` means it is ready. Open PREVIEW for the student.

Tell them, in your own words:

- This is the new version, on your computer only. Nobody else can open this address.
- Your live site still shows the old version.
- Look at what changed, then click through the rest of the page to see nothing else moved.

Ask: "Does it look right? Say yes and I'll put it online, or tell me what to change." Wait for the answer. The preview keeps running while you wait.

- **They want something changed.** Make the change and tell them to refresh the preview page. Ask again.
- **Yes.** Stop the preview, then go on to part 3.
- **Not now.** Stop the preview. Tell them the change is saved on their computer and is not online, and that they can say "update my site" whenever they are ready. That starts this skill from the top, and the preview comes back.

To stop the preview, stop the background command the way your tool does it. Then run the check above again: it prints `000` when the preview is gone. Always stop it before you finish, whatever the answer.

If there is no "Ready on" line after 30 seconds, stop the background command and tell the student the preview did not start. Open the site's `index.html` from SITE directly, with `open` on a Mac or `start` on Windows, and tell them pictures and styles can look a little different in this view than on the live site. Then ask the same question.

### 3. Online

Only after the student's yes to this version. If you skipped the preview because they told you to, that was the yes.

Say: "I'm sending the new version to Cloudflare, then saving it to GitHub."

Run this from PROJECT.

    wrangler deploy 2>&1 | tail -8

Then check the address from the output answers:

    curl -s -o /dev/null -w '%{http_code}\n' ADDRESS

`200` means the new version is up. Tell the student to refresh the page. If they still see the old version, their browser kept a copy: have them open the address in a private window.

### 4. Saved

If step 1 said "no history yet", do step 5 instead of this.

    git add -A && git commit -q -m "MESSAGE"; git push -q 2>&1 | tail -3; git log --oneline -1

If it says there is nothing to commit, nothing changed since the last save, and that is fine. If the push is rejected because GitHub has changes this computer does not, run `git pull --rebase` and then the push again.

## Your Cloudflare name

Only if Cloudflare said "You need to register a workers.dev subdomain".

Say: "Every site you put online gets an address that ends in a name belonging to your account, like my-site.YOUR-NAME.workers.dev. A new account has to open one page before that name exists. It takes a minute."

Open `https://dash.cloudflare.com/?to=/:account/workers-and-pages` and tell them:

1. Sign in if it asks. You don't need to create anything on this page.
2. Look for the word Subdomain. It is usually on the right side, with a name next to it that ends in `.workers.dev`.
3. If a name is already there, Cloudflare picked one for you. You can keep it, or click it and change it.
4. If it asks you to set one up, type a name with lowercase letters, numbers or dashes, and save it.
5. This name is public and is part of the address of every site you make.
6. Tell me the name that ends in `.workers.dev`.

Do not send them to a link that ends in `/workers/onboarding`. Wrangler suggests it and it leads to a "page not found".

## If they ask

- **Take the site down.** Confirm the name and that they want it gone, then run `wrangler delete --name SITE-NAME`. The files on their computer and on GitHub stay.
- **Make the GitHub copy public.** `gh repo edit --visibility public --accept-visibility-change-consequences`, only on their yes, and tell them every file in the folder becomes readable by anyone, including notes that are not on the site.
- **The page is blank or says "not found".** The folder Cloudflare was given has no `index.html` directly in it. Check the `directory` line in `wrangler.jsonc` points at the folder that holds it, then put it online again.
- **Pictures or styles are missing online but fine on their computer.** Usually a link in the page points at a file by its place on their computer. Find the broken links, fix them to point at the file's place inside the site folder, and put it online again.
- **Their own address, like mysite.com.** That is possible and is not part of this. Do not set it up here. Tell them one fact so they can plan: a domain bought at Cloudflare can be connected directly. A domain bought somewhere else (Squarespace, GoDaddy, Namecheap) has to have its nameservers moved to Cloudflare first, which changes who runs the domain's settings. Tell them to ask in class before they buy or move anything.
