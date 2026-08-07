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

---

## Troubleshooting

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
