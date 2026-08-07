# Publishing the dashboard to GitHub Pages

Repo name: **`bmt`** · Account: **eduardomansi** · Auth: **fine-grained personal access token**
**Final URL: https://eduardomansi.github.io/bmt/**

> Everything below is filled in for your account — copy-paste as-is, no placeholders to swap.
>
> This is a **project site** and is completely separate from your existing user site at `eduardomansi.github.io`. That one is untouched; you can have unlimited project sites alongside it.

**Only this `site/` folder gets published.** The parent folder — `tracker.md`, `backtest.md`, the dated reports — stays private on your Mac. Keep it that way: run `git init` *inside* `site/`, never in the parent.

---

## Step 1 — Create the repo

Go to **https://github.com/new**

- **Repository name:** `bmt`
- **Public** — Pages needs a public repo on the free plan
- **Do not** tick "Add a README", "Add .gitignore", or choose a license. The repo must be completely empty or the first push will be rejected.

Click **Create repository**, then leave the page open.

---

## Easier alternative — skip Steps 2 and 3 entirely

Manual tokens are the fiddliest way to do this. GitHub CLI signs in via browser and configures git for you — no token to create, scope, store or renew:

```
brew install gh          # or the installer from cli.github.com
gh auth login
```

Answer: **GitHub.com** → **HTTPS** → **Authenticate Git with your GitHub credentials? Yes** → **Login with a web browser**. Paste the one-time code it shows, authorize in the browser, done.

Then, from this folder:

```
gh repo create bmt --public --source=. --push
```

That creates the repo, sets the remote, and pushes — replacing Steps 1, 2 and 3. Go straight to Step 4.

If the repo already exists, just `git push -u origin main` instead.

> Already have a stale credential cached? Erase it first or git will silently reuse it:
> `git credential-osxkeychain erase`, then `protocol=https`, `host=github.com`, Return twice.

---

## Step 2 — Generate the token

Go to **https://github.com/settings/tokens?type=beta**
(or: avatar → Settings → Developer settings → Personal access tokens → **Fine-grained tokens**)

Click **Generate new token**, then set:

| Field | Value |
|---|---|
| **Token name** | `bmt pages push` |
| **Expiration** | Your call — 90 days is a sensible default. Note the date; pushes fail silently-ish when it lapses. |
| **Repository access** | **Only select repositories** → pick **`bmt`** |
| **Permissions** → Repository permissions → **Contents** | **Read and write** |

Leave every other permission at "No access". *Metadata: Read-only* gets added automatically and is required — that's expected.

Click **Generate token**, then **copy it**. GitHub shows it exactly once.

> **Paste it only into your Terminal when git asks for a password.** Don't put it in this file, don't commit it, and don't send it to me in chat — I won't accept it, and you'd have to revoke it. If it ever does leak, revoke it on that same settings page; it only has write access to this one repo, which is why we scoped it that way.

---

## Step 3 — Push

Tell git to remember the token in your macOS Keychain so you're not re-pasting it daily:

```
git config --global credential.helper osxkeychain
```

Then publish the folder:

```
cd ~/Documents/Claude/Scheduled/stockedup-big-money-scan/site
git init -b main
git add -A
git commit -m "dashboard"
git remote add origin https://github.com/eduardomansi/bmt.git
git push -u origin main
```

When prompted:
- **Username:** `eduardomansi`
- **Password:** paste the **token** (not your account password — GitHub stopped accepting those over HTTPS)

Nothing will appear as you paste. That's normal; press Return.

---

## Step 4 — Turn on Pages

In the repo: **Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder `/ (root)` → Save.**

Your URL: **`https://eduardomansi.github.io/bmt/`**

First build takes a minute or two. Open it on your phone and use **Share → Add to Home Screen** — it launches like an app.

---

## After each daily scan

The scheduled task rewrites `index.html` here. To publish:

```
cd ~/Documents/Claude/Scheduled/stockedup-big-money-scan/site
git add -A && git commit -m "update $(date +%F)" && git push
```

Worth saving as a shell alias. Add to `~/.zshrc`:

```
alias bmt='cd ~/Documents/Claude/Scheduled/stockedup-big-money-scan/site && git add -A && git commit -m "update $(date +%F)" && git push'
```

Then it's just `bmt` in a terminal. Run `source ~/.zshrc` once to pick it up.

### Fully automatic — recommended (one-time, ~1 minute)

The scan itself cannot push: it runs in an isolated sandbox with no access to your Keychain. Instead, a **launchd agent on your Mac watches `site/index.html` and pushes the moment the scan rewrites it.** Event-driven, so it publishes right after every run rather than on a guessed timer.

Two files in the parent folder do this: `push.sh` (the git commands) and `com.eduardomansi.bmt-push.plist` (the watcher).

Install once:

```
cd ~/Documents/Claude/Scheduled/stockedup-big-money-scan
chmod +x push.sh
cp com.eduardomansi.bmt-push.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.eduardomansi.bmt-push.plist
```

Test it without waiting for the next scan:

```
touch site/index.html
sleep 5 && cat push.log
```

You should see a timestamp and either `pushed OK` or `no changes, skipping`.

**Why launchd rather than cron:** it triggers on the file changing instead of at a fixed time, so it can't miss a late-finishing run, and LaunchAgents run inside your GUI session where the Keychain is unlocked — cron is flakier about that.

Notes:
- Everything is logged to `push.log`. Check there first if the site looks stale.
- Empty runs are skipped rather than making empty commits.
- If your token expires, `push.log` says so; run `git push` manually once to re-authenticate.
- To disable: `launchctl unload ~/Library/LaunchAgents/com.eduardomansi.bmt-push.plist`
- `push.sh`, `push.log` and the plist live in the **parent** folder, not `site/`, so they never get published.

---

## Troubleshooting

**`Permission to eduardomansi/bmt.git denied` / error 403** — authentication worked, authorization didn't. Two causes, often together:

1. *Git is using an old credential and never prompted you.* If you've pushed to GitHub from this Mac before, Keychain already holds a token — it authenticates fine but has no write access to `bmt`. Clear it:
   ```
   git credential-osxkeychain erase
   protocol=https
   host=github.com
   ```
   then Return twice on a blank line.
2. *The token doesn't cover this repo.* Fine-grained tokens set to "Only select repositories" only cover repos that **existed when the token was created**. Make the repo first, then the token. Verify under Repository access that `bmt` is ticked, and that **Contents = Read and write** — a missing or read-only Contents permission produces exactly this 403.

**No prompt for username/password at all** — same cause as (1) above: a cached Keychain credential is being used silently. Erase it and push again.

**Push rejected, "updates were rejected"** — the repo wasn't empty. Either delete and recreate it without a README, or run `git pull --rebase origin main` then push again.

**Auth suddenly fails** — the token expired. Generate a new one (Step 2), then clear the stale Keychain entry so git stops reusing it:

```
git credential-osxkeychain erase
host=github.com
protocol=https
```

Press Return twice after the last line. Next push will prompt for the new token.

**Page 404s after enabling Pages** — give it 2–3 minutes for the first build. Check the **Actions** tab for a failed deploy. Confirm `index.html` is at the repo root, not nested in a folder.

**Changes don't show** — hard-refresh; Pages caches. On iOS Safari, close the tab and reopen.

---

## Worth knowing

- **The site is public.** Anyone with the URL can read it. `noindex` keeps it out of search results, but that's obscurity, not access control. Nothing here is sensitive today — tickers, strikes and expiries derived from a public YouTube channel. **If you ever add real position sizes or account details, move to a private host first** (Cloudflare Pages with Access is the free option).
- **Prices are a snapshot** from the moment of the daily scan, and the page says so. There is deliberately no API key in this file — a key on a public page is a key you've given away.
- **`.nojekyll`** stops GitHub running the site through Jekyll. Leave it.
- **Don't move `index.html`** out of this folder, or the task's Step 8b will lose track of it.
