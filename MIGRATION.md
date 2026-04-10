# Repository Consolidation Plan: LAF-PUBLIC & LAF-PRIVATE

## Overview

Logan currently has several scattered public repositories. The goal is to consolidate everything into two deliberately controlled repositories:

| Repo | Purpose |
|------|---------|
| **LAF-PUBLIC** | All publicly shareable projects and work |
| **LAF-PRIVATE** | Private, sensitive, or work-in-progress content |

A **GitHub Organization** named **LAF-US** is recommended to host both repos and enable cleaner access control, team collaboration, and branding. See [GitHub's guidance on Organizations](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations).

---

## Current Repositories

| Repo | Visibility | Description | Target |
|------|-----------|-------------|--------|
| [loganfinney27.github.io](https://github.com/loganfinney27/loganfinney27.github.io) | Public | GitHub Pages personal site | LAF-PUBLIC |
| [THE-GEMSTONE](https://github.com/loganfinney27/THE-GEMSTONE) | Public | Independent publication about Idaho | LAF-PUBLIC |
| [IR-Court-Tracker](https://github.com/loganfinney27/IR-Court-Tracker) | Public | Idaho Reports court tracker (Python) | LAF-PUBLIC |
| [IDEX_Artifacts](https://github.com/loganfinney27/IDEX_Artifacts) | Public | Idaho Experience "Our American Artifacts" (HTML) | LAF-PUBLIC |
| [PyTutorial](https://github.com/loganfinney27/PyTutorial) | Public | Python learning playground (this repo) | LAF-PUBLIC |
| [IDAHO-VAULT](https://github.com/loganfinney27/IDAHO-VAULT) | Public | Obsidian master archive | LAF-PRIVATE |

---

## Target Structure

All content from the current public repos is merged **flat** into the root of `LAF-PUBLIC` — no per-project subdirectories. The goal is to break down silos entirely, not recreate them internally.

### LAF-PUBLIC

```
LAF-PUBLIC/
├── README.md
└── (all files from loganfinney27.github.io, THE-GEMSTONE, IR-Court-Tracker,
    IDEX_Artifacts, and PyTutorial merged at root level)
```

> **Note:** If two source repos contain a file with the same name, resolve the conflict manually during migration before committing.

### LAF-PRIVATE

```
LAF-PRIVATE/
├── README.md
└── (all files from IDAHO-VAULT merged at root level)
```

---

## Pre-flight Checks: How the Existing Repos Interact

Before merging anything, these cross-repo dependencies must be understood and explicitly resolved.

### 1. THE-GEMSTONE is a Quartz 4.0 fork — not plain content

THE-GEMSTONE is NOT a simple collection of documents. It is a full fork of the [Quartz](https://quartz.jzhao.xyz/) static site generator (v4). Its root contains an entire Node.js build system:

| File / Dir | Purpose |
|---|---|
| `quartz/` | Quartz framework source (TypeScript, plugins, styles) |
| `quartz.config.ts` | Site configuration — sets `baseUrl: "thegemstone.org"` |
| `quartz.layout.ts` | Layout customization |
| `package.json` / `package-lock.json` | Node.js dependencies |
| `tsconfig.json`, `globals.d.ts`, `index.d.ts` | TypeScript config |
| `Dockerfile` | Container build for local preview |
| `content/` | Obsidian Markdown source notes (the actual publication content) |
| `.github/workflows/deploy.yml` | CI pipeline: builds Quartz → deploys to GitHub Pages |

When merged flat into LAF-PUBLIC, all of these files land at the repo root and will conflict with files from other source repos (e.g., multiple `README.md`, `package.json`, `.gitignore`, `.github/workflows/` files).

### 2. THE-GEMSTONE self-deploys to GitHub Pages with a custom domain

THE-GEMSTONE's `.github/workflows/deploy.yml` runs on every push to `main`:
1. Checks out the repo
2. Runs `npm ci && npx quartz build` → outputs to `public/`
3. Uploads the `public/` artifact and deploys it via `actions/deploy-pages`

The site is served at the custom domain **`thegemstone.org`** (configured in `quartz.config.ts`).

**Impact on migration:** Moving THE-GEMSTONE into LAF-PUBLIC means:
- GitHub Pages must be enabled on **LAF-PUBLIC** in repo Settings → Pages → Source: GitHub Actions
- The `CNAME` / custom domain (`thegemstone.org`) must be re-configured on LAF-PUBLIC
- `quartz.config.ts` → `baseUrl` value may need updating if the domain changes
- The deploy workflow must still exist at `.github/workflows/deploy.yml` in LAF-PUBLIC after the merge (it will come in via the flat merge — confirm there are no conflicts with other workflows)

### 3. `loganfinney27.github.io` is independent of THE-GEMSTONE

`loganfinney27.github.io` is a **separate, standalone repo** — it contains only a hand-written `index.html` ("Hello World, I'm hosted with GitHub Pages") and a `README.md`. It does **not** serve as the build target for THE-GEMSTONE; that site deploys to its own GitHub Pages environment under `thegemstone.org`.

Both repos can be merged into LAF-PUBLIC independently. The `index.html` from `loganfinney27.github.io` will conflict with any root-level HTML from other repos — resolve manually.

### 4. Filename conflict map (known collisions before merging)

| Filename | Repos that contain it |
|---|---|
| `README.md` | **All 5 public repos** — must be manually merged into one |
| `.gitignore` | THE-GEMSTONE, IR-Court-Tracker, possibly others |
| `package.json` / `package-lock.json` | THE-GEMSTONE (others TBC) |
| `.github/workflows/*.yml` | THE-GEMSTONE (`deploy.yml`, `auto-merge.yml`) — check others |
| `index.html` | `loganfinney27.github.io` — check IDEX_Artifacts |

Resolve each conflict before committing each repo merge.

### 5. Quartz upstream sync — ongoing consideration

THE-GEMSTONE was set up to receive upstream patches from `jackyzha0/quartz`. After consolidation, the upstream remote in THE-GEMSTONE's Quartz fork history no longer exists as a separate repo. If upstream Quartz updates are needed in the future, they must be cherry-picked or merged directly into LAF-PUBLIC from the upstream Quartz repo.

---

## Migration Steps

### Step 1: Create a GitHub Organization (optional but recommended)

1. Go to [github.com/organizations/new](https://github.com/organizations/new) and create a free organization named **`LAF-US`**.
2. Transfer ownership of existing repos to the org, or create the new consolidated repos directly under the org.

> **Why an Organization?** Organizations allow granular team permissions, separate billing, and a professional namespace. All repos remain under your control. See the [community discussion](https://github.com/orgs/community/discussions/143759) for more context.

### Step 2: Create the LAF-PUBLIC repository

```bash
# On GitHub.com (or via gh CLI) — create under the LAF-US organization:
gh repo create LAF-US/LAF-PUBLIC --public --description "All public-facing LAF projects"
```

### Step 3: Migrate each public repo into LAF-PUBLIC (flat — no subdirectories)

Merge each existing repo directly into the root of `LAF-PUBLIC`, preserving commit history.
See the **Pre-flight Checks** section above for known filename conflicts before starting.

```bash
# Clone the new empty LAF-PUBLIC repo
git clone https://github.com/LAF-US/LAF-PUBLIC.git
cd LAF-PUBLIC

# --- PyTutorial ---
git remote add py-tutorial https://github.com/loganfinney27/PyTutorial.git
git fetch py-tutorial
git merge --allow-unrelated-histories py-tutorial/main
# Resolve conflicts (e.g. README.md), then:
git commit -m "chore: merge PyTutorial into LAF-PUBLIC"

# --- THE-GEMSTONE (Quartz fork — brings full build system) ---
git remote add the-gemstone https://github.com/loganfinney27/THE-GEMSTONE.git
git fetch the-gemstone
git merge --allow-unrelated-histories the-gemstone/main
# Resolve conflicts (README.md, .gitignore, package.json, .github/workflows/, etc.)
# Preserve quartz.config.ts, quartz.layout.ts, quartz/, content/ intact.
git commit -m "chore: merge THE-GEMSTONE (Quartz) into LAF-PUBLIC"

# --- IR-Court-Tracker ---
git remote add ir-court-tracker https://github.com/loganfinney27/IR-Court-Tracker.git
git fetch ir-court-tracker
git merge --allow-unrelated-histories ir-court-tracker/main
git commit -m "chore: merge IR-Court-Tracker into LAF-PUBLIC"

# --- IDEX_Artifacts ---
git remote add idex-artifacts https://github.com/loganfinney27/IDEX_Artifacts.git
git fetch idex-artifacts
git merge --allow-unrelated-histories idex-artifacts/main
git commit -m "chore: merge IDEX_Artifacts into LAF-PUBLIC"

# --- loganfinney27.github.io ---
git remote add github-pages https://github.com/loganfinney27/loganfinney27.github.io.git
git fetch github-pages
git merge --allow-unrelated-histories github-pages/main
# index.html conflicts with any root HTML — merge content manually.
git commit -m "chore: merge loganfinney27.github.io into LAF-PUBLIC"

git push origin main
```

> **Conflict resolution:** When Git flags a merge conflict, manually combine the file content, run `git add <file>`, then commit before moving to the next repo. The README.md conflict will occur on every merge — keep building a single consolidated README.

### Step 3a: Re-enable GitHub Pages and custom domain on LAF-PUBLIC

After the merge, THE-GEMSTONE's Quartz deployment pipeline is now in LAF-PUBLIC. To restore the live site:

1. In **LAF-PUBLIC repo Settings → Pages**, set Source to **GitHub Actions**.
2. Re-enter the custom domain **`thegemstone.org`** in the Custom domain field and save.
3. Update your DNS provider: point the `CNAME` (or `A` records) for `thegemstone.org` to `laf-us.github.io` (replacing the old `loganfinney27.github.io` target).
4. Confirm `quartz.config.ts` still has `baseUrl: "thegemstone.org"` — update if the domain changes.
5. Push a commit to trigger the deploy workflow and verify the site loads at `thegemstone.org`.

### Step 4: Create the LAF-PRIVATE repository

```bash
gh repo create LAF-US/LAF-PRIVATE --private --description "Private LAF projects and archives"
```

Then merge IDAHO-VAULT directly into the root of `LAF-PRIVATE`:

```bash
git clone https://github.com/LAF-US/LAF-PRIVATE.git
cd LAF-PRIVATE
git remote add idaho-vault https://github.com/loganfinney27/IDAHO-VAULT.git
git fetch idaho-vault
git merge --allow-unrelated-histories idaho-vault/main
git commit -m "chore: merge IDAHO-VAULT into LAF-PRIVATE"
git push origin main
```

### Step 5: Archive the old repositories

Once migration is complete and verified:

1. Navigate to each old repo → **Settings** → **Danger Zone** → **Archive this repository**.
2. This makes them read-only and clearly marks them as superseded — without deleting history.

### Step 6: Update cross-references

- Update any links in README files, blog posts, or websites that point to the old repo URLs.
- Add a redirect notice to each archived repo's README pointing to the new consolidated location.

---

## Checklist

- [ ] Create GitHub Organization **LAF-US**
- [ ] **Pre-flight:** Review conflict map (README.md, package.json, .gitignore, workflows, index.html)
- [ ] **Pre-flight:** Note THE-GEMSTONE custom domain (`thegemstone.org`) and DNS settings
- [ ] Create `LAF-PUBLIC` repo (public)
- [ ] Merge `PyTutorial` → `LAF-PUBLIC` (flat)
- [ ] Merge `THE-GEMSTONE` (Quartz fork) → `LAF-PUBLIC` (flat) — resolve build-system file conflicts
- [ ] Merge `IR-Court-Tracker` → `LAF-PUBLIC` (flat)
- [ ] Merge `IDEX_Artifacts` → `LAF-PUBLIC` (flat)
- [ ] Merge `loganfinney27.github.io` → `LAF-PUBLIC` (flat)
- [ ] Enable GitHub Pages on LAF-PUBLIC (Source: GitHub Actions)
- [ ] Re-configure custom domain `thegemstone.org` on LAF-PUBLIC and update DNS
- [ ] Verify Quartz site builds and deploys from LAF-PUBLIC
- [ ] Create `LAF-PRIVATE` repo (private)
- [ ] Merge `IDAHO-VAULT` → `LAF-PRIVATE` (flat)
- [ ] Archive all original repositories
- [ ] Update any external links/references

---

## References

- [GitHub Docs: About Organizations](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)
- [GitHub Docs: Archiving a repository](https://docs.github.com/en/repositories/archiving-a-github-repository/archiving-repositories)
- [GitHub Docs: git subtree](https://git-scm.com/book/en/v2/Git-Tools-Subtree-Merging)
- [Community Discussion: Organizing repos](https://github.com/orgs/community/discussions/143759)
