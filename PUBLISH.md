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

## Publishing — already automatic (working as of 2026-08-07)

**You don't need to do anything after a scan.** A launchd agent watches `~/bmt/site/index.html` and pushes within seconds of the scheduled scan rewriting it. Event-driven, so it can't miss a run that finishes late.

The chain:

```
scan writes ~/bmt/site/index.html
   → launchd agent com.eduardomansi.bmt-push fires
   → ~/bmt/push.sh commits and pushes
   → GitHub Pages redeploys (~1 min)
```

### ⚑ Why this folder is at ~/bmt and not under ~/Documents

`~/Documents` is **TCC-protected** on macOS. Processes spawned by launchd are denied access to it and die with `Operation not permitted` — before they can even write a log, which makes it look like nothing ran at all. Granting Full Disk Access to the *script* doesn't help either: macOS attributes the permission to the interpreter (`/bin/bash`), not the `.sh` file.

The options were to grant FDA to `/bin/bash` — which hands full disk access to every shell script on the machine — or to move the repo somewhere unprotected. We moved it. **Do not move this repo back under `~/Documents`, `~/Desktop` or `~/Downloads`**, or auto-push silently breaks again.

The scan's data files (`tracker.md`, `backtest.md`, reports) still live in `~/Documents/Claude/Scheduled/stockedup-big-money-scan/`. That's fine — only the *publishing* half needed to move.

### Layout

```
~/bmt/
├── push.sh        ← git commands, run by launchd
├── push.log       ← every run appends here
├── com.eduardomansi.bmt-push.plist
└── site/          ← the git repo; this is what GitHub serves
    ├── index.html
    ├── .nojekyll
    └── PUBLISH.md
```

Script, log and plist sit **outside** `site/`, so they're never published.

### If you ever need to reinstall the agent

```
chmod +x ~/bmt/push.sh
cp ~/bmt/com.eduardomansi.bmt-push.plist ~/Library/LaunchAgents/
launchctl bootout gui/$(id -u)/com.eduardomansi.bmt-push 2>/dev/null
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.eduardomansi.bmt-push.plist
```

Use `bootstrap`, not `launchctl load` — the latter is deprecated and fails silently.

Test without waiting for a scan:

```
touch ~/bmt/site/index.html && sleep 8 && tail -4 ~/bmt/push.log
```

A fresh timestamp means it fired. `no changes, skipping` is a pass — it proves the agent ran and reached git.

### Manual push, if you want one

```
cd ~/bmt/site && git add -A && git commit -m "update" && git push
```

### Notes

- **`push.log` is the first place to look** if the site seems stale.
- Empty runs are skipped rather than making empty commits.
- Token expiry shows up in `push.log`; run the manual push once to re-authenticate.
- `/tmp/bmt-push.err` captures launchd-level errors, but **it accumulates and is never cleared** — old entries linger and look alarming. `rm /tmp/bmt-push.err` before testing.
- Disable with: `launchctl bootout gui/$(id -u)/com.eduardomansi.bmt-push`

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

**`Operation not permitted` in `/tmp/bmt-push.err`** — the repo has been moved back into a TCC-protected folder (`~/Documents`, `~/Desktop`, `~/Downloads`). Move it back to `~/bmt` and reinstall the agent. See the section above; this is not fixable with a Full Disk Access grant on the script.

**Site stale but `push.log` shows `pushed OK`** — GitHub Pages caches. Hard-refresh, or on iOS Safari close the tab and reopen. Check the repo's **Actions** tab for a failed deploy.

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
