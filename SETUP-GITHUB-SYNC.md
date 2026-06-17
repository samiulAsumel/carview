# CarView — GitHub Sync Setup (Banglish)

Firebase soriye GitHub + Cloudflare Worker-e migrate kora hoyeche.

- **App** (`carview` repo) -> Cloudflare Pages -> carview.pages.dev
- **Data** (`carview-data` repo, PRIVATE) -> sudhu `data.json`
- **Worker** (`carview-proxy.sa-sumel91.workers.dev`) -> app ar private repo-r majhe proxy. GitHub token sudhu Worker-er bhitore thake, browser-e kokhono ashe na.

```
Viewer/Browser  --GET-->  Worker  --(token)-->  GitHub (read data.json)
You (logged in) --PUT-->  Worker  --(token)-->  GitHub (commit data.json)
```

---

## Ekbar-er setup (15 min)

### 1. GitHub token (PAT) banao

1. GitHub -> Settings -> Developer settings -> **Fine-grained tokens** -> "Generate new token".
2. **Repository access** -> "Only select repositories" -> `carview-data` select koro.
3. **Permissions** -> Repository permissions -> **Contents: Read and write**.
4. Generate kore token-ta copy koro (`github_pat_...`). Eta ekbar-i dekhabe.

### 2. Worker-e code ar secret boshao

1. Cloudflare dashboard -> Workers & Pages -> `carview-proxy` open koro (already ache).
2. Edit code -> purono code muche **`worker.js`**-er pura code paste koro -> Deploy.
3. Settings -> **Variables and Secrets** -> ei duita **Secret** add koro:
   - `GITHUB_TOKEN`  = step-1 er PAT
   - `WRITE_PASSWORD` = tomar app-er login password
4. Deploy / Save.

> Type "Secret" rakhba (Text na), jate value lukano thake.

### 3. Test

- Browser-e `https://carview-proxy.sa-sumel91.workers.dev` kholo -> data.json-er JSON dekha gele READ thik ache.
- carview.pages.dev -> login -> ekta entry change kore Save -> "Saved to cloud" elে WRITE thik ache.
- GitHub `carview-data` -> Commits -> notun "Update data ..." commit dekhbe.

---

## Daily use

- Tumi jekono device/browser theke id+password diye login korba -> auto-save cloud-e jabe.
- Onno keu (login chara) sudhu dekhte parbe, edit/save parbe na.
- **Protita save = ekta GitHub commit.** Tai purono kono version kokhono harabe na -- GitHub-er Commit history theke jekono purono data fire ana jay.

## Notun feature

GitHub history kaje lagiye 3-ta notun feature add kora hoyeche (Settings → Cloud Sync panel-e):

- **Version History (📜)** — purono protita save-er list. Ek click-e jekono purono version **Restore** kora jay. Restore korle eta notun commit hisebe boshe, purono kichu mochena.
- **Overwrite protection** — duita device theke edit korle, purono data overwrite hobar age warning dey ("reload kore abar Save korun").
- **Last saved** — sesh kobe cloud-e save hoyeche seta dekhay.

## Khoyal rakho

- **Password change korle** Worker-er `WRITE_PASSWORD` secret-o new password-e update korte hobe, na hole save reject hobe.
- Sob free tier-er bhitore: Worker 100k req/din free, GitHub repo free, Pages free.

---

## Ki ki change holo (technical)

| File | Change |
|------|--------|
| `app.js` | Firebase config/init -> `GITHUB_CONFIG` + Worker. `loadFromFirebase` = Worker GET. `saveToFirebase` = Worker PUT (+ `X-Write-Key`). Login-e `writeAuth` derive. Password-change ar connected-status-er Firebase call soriye Worker/static kora hoyeche. |
| `index.html` | CSP `connect-src` -> Worker domain. UI label Firebase -> GitHub. |
| `service-worker.js` | Cache `v4` -> `v5`. Cloud data network-first routing Worker host-e. |
| `worker.js` | **Notun** — Cloudflare Worker (GET read, `?history=1` commit list, `?at=<sha>` old version, PUT write with auth + overwrite-conflict check). |
