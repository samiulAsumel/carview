# CarView — GitHub Sync Setup

Cloud sync was migrated from Firebase to GitHub + a Cloudflare Worker.

- **App** (`carview` repo) → Cloudflare Pages → carview.pages.dev
- **Data** (`carview-data` repo, PRIVATE) → a single `data.json`
- **Worker** (`carview-proxy.sa-sumel91.workers.dev`) → a proxy between the app and the private repo. The GitHub token lives only inside the Worker and is never exposed to the browser.

```
Viewer/Browser  --GET-->  Worker  --(token)-->  GitHub (read data.json)
You (logged in) --PUT-->  Worker  --(token)-->  GitHub (commit data.json)
```

---

## One-time setup (15 min)

### 1. Create a GitHub token (PAT)

1. GitHub → Settings → Developer settings → **Fine-grained tokens** → "Generate new token".
2. **Repository access** → "Only select repositories" → select `carview-data`.
3. **Permissions** → Repository permissions → **Contents: Read and write**.
4. Generate and copy the token (`github_pat_...`). It is shown only once.

### 2. Add the code and secrets to the Worker

1. Cloudflare dashboard → Workers & Pages → open `carview-proxy` (already created).
2. Edit code → delete the old code → paste the full contents of **`worker/worker.js`** → Deploy.
3. Settings → **Variables and Secrets** → add these two as **Secret**:
   - `GITHUB_TOKEN`  = the PAT from step 1
   - `WRITE_PASSWORD` = your app login password
4. Deploy / Save.

> Set the type to "Secret" (not "Text") so the values stay hidden.

### 3. Test

- Open `https://carview-proxy.sa-sumel91.workers.dev` in a browser — if you see the JSON of `data.json`, READ works.
- Open carview.pages.dev → log in → change an entry and Save → if you see "Saved to cloud", WRITE works.
- GitHub `carview-data` → Commits → you should see a new "Update data ..." commit.

---

## Daily use

- Log in with your id and password from any device/browser → changes auto-save to the cloud.
- Anyone else (not logged in) can only view; they cannot edit or save.
- **Every save is one GitHub commit.** No version is ever lost — any previous version can be restored from the GitHub commit history.

## Features

Three features built on top of the GitHub history (Settings → Cloud Sync panel):

- **Version History (📜)** — a list of every previous save. Any version can be **Restored** with one click. A restore is committed as a new version, so nothing is deleted.
- **Overwrite protection** — if two devices edit at once, you are warned before older data is overwritten ("reload the page and save again").
- **Last saved** — shows when the data was last saved to the cloud.

## Notes

- **If you change the password**, also update the Worker's `WRITE_PASSWORD` secret to the new password, otherwise saves are rejected.
- Everything stays within free tiers: Worker 100k requests/day free, GitHub repo free, Pages free.
- The fine-grained PAT has an expiry — when it expires, saves fall back to device-only. Renew it in the Worker's `GITHUB_TOKEN` secret.

---

## What changed (technical)

| File | Change |
|------|--------|
| `src/app.js` | Firebase config/init → `GITHUB_CONFIG` + Worker. `loadFromFirebase` = Worker GET. `saveToFirebase` = Worker PUT (+ `X-Write-Key`). Login derives `writeAuth`. Firebase calls for password-change and connected-status replaced with Worker/static logic. |
| `index.html` | CSP `connect-src` → Worker domain. UI labels Firebase → GitHub. |
| `service-worker.js` | Cloud data uses network-first routing to the Worker host. (Bump `CACHE_NAME` on every code change.) |
| `worker/worker.js` | **New** — Cloudflare Worker (GET read, `?history=1` commit list, `?at=<sha>` old version, PUT write with auth + overwrite-conflict check). |
