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
| [loganfinney27.github.io](https://github.com/loganfinney27/loganfinney27.github.io) | Public | Standalone "Hello World" GitHub Pages site (`index.html` + `README.md`) | LAF-PUBLIC |
| [THE-GEMSTONE](https://github.com/loganfinney27/THE-GEMSTONE) | Public | Quartz 4.0 static site fork — independent Idaho publication at `thegemstone.org`, self-deploys via GitHub Actions | LAF-PUBLIC |
| [IR-Court-Tracker](https://github.com/loganfinney27/IR-Court-Tracker) | Public | Python scraper tracking Idaho federal court cases via CourtListener; daily GitHub Actions workflow auto-commits `output.csv` | LAF-PUBLIC |
| [IDEX_Artifacts](https://github.com/loganfinney27/IDEX_Artifacts) | Public | Idaho Experience "Our American Memories" TV production artifacts (RESEARCH, MEDIA, PROJECTS, LOGISTICS, SCRIPTS) | LAF-PUBLIC |
| [PyTutorial](https://github.com/loganfinney27/PyTutorial) | Public | Python learning playground with Jupyter notebooks and scripts | LAF-PUBLIC |
| [IDAHO-VAULT](https://github.com/loganfinney27/IDAHO-VAULT) | Public* | Obsidian knowledge vault + master AI agent governance hub (1,000+ files, 20+ active workflows) — **currently public, must be moved to PRIVATE** | LAF-PRIVATE |

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
| `output.csv` | IR-Court-Tracker (actively written by daily workflow), PyTutorial (empty file) — decide which takes precedence |
| `requirements.txt` | IR-Court-Tracker, PyTutorial — merge dependency lists, check for version conflicts |
| `.gitignore` | THE-GEMSTONE, IR-Court-Tracker, IDEX_Artifacts, PyTutorial — merge into one combined `.gitignore` |
| `.gitattributes` | THE-GEMSTONE, IR-Court-Tracker, PyTutorial |
| `package.json` / `package-lock.json` | THE-GEMSTONE (Quartz build system — others TBC) |
| `.github/workflows/*.yml` | THE-GEMSTONE (`deploy.yml`, `auto-merge.yml`), IR-Court-Tracker (`python-run.yml`) — check IDEX_Artifacts |
| `index.html` | `loganfinney27.github.io` — check IDEX_Artifacts |
| `AGENTS.md` | IR-Court-Tracker — check other repos before merging |

Resolve each conflict before committing each repo merge.

### 5. Quartz upstream sync — ongoing consideration

THE-GEMSTONE was set up to receive upstream patches from `jackyzha0/quartz`. After consolidation, the upstream remote in THE-GEMSTONE's Quartz fork history no longer exists as a separate repo. If upstream Quartz updates are needed in the future, they must be cherry-picked or merged directly into LAF-PUBLIC from the upstream Quartz repo.

---

## IDAHO-VAULT: Context and Broader Goals

IDAHO-VAULT is far more than a personal notes archive. Understanding what it contains is essential for executing a safe migration.

### What it is

IDAHO-VAULT is a dual-purpose repository combining two deeply intertwined systems:

1. **An Obsidian.md personal knowledge vault** — 1,000+ Markdown files organized mostly flat at the repo root. Topics span Idaho legislation, Idaho journalism research, court case notes, production artifacts from Idaho Public Television, personal organization, and reference notes.

2. **The master governance hub for the "UNIFIED (US) SWARM"** — a multi-agent AI infrastructure Logan has built to assist his journalism and knowledge-management work. This is the brain that all other repos point back to.

### The UNIFIED (US) SWARM

IDAHO-VAULT is the single source of truth for an active swarm of AI agents working across Logan's repositories. Key governance files:

| File | Purpose |
|---|---|
| `CONSTITUTION.md` | Root canonical constitution — binding governance for all agents |
| `AGENTS.md` (root) | Cross-tool pointer auto-loaded by Codex CLI, Copilot, and Qodo |
| `!/AGENTS.md` | Canonical narrative registry — agent roster, capability tiers, lane rules |
| `swarm.json` | Machine-readable agent registry; contains hardcoded repo URLs |
| `DECISIONS.md` | Logan-approved decisions log |
| `!README.md` | Touchstone Tree — live orienting doctrine |
| `!/agents.json` | Generated bootstrap index for local agent startup |
| `!/agent.sh` | Local bootstrap entrypoint for agents |

**Agent roster (from `!/AGENTS.md`):**

| Agent | Persona | Role |
|---|---|---|
| Claude Code | The Abhorsen | Authority: Code — executor |
| Gemini CLI | The Vault Advisor | Direct Write — interpreter |
| OpenAI Codex | The Lexicographer | Scripting/Automation |
| GitHub Copilot | The Clerk | Multi-Repo Admin |
| Grok | The Ironist | Read/Analysis |
| DeepSeek | The Analyst | Advisory |
| Perplexity | The Scout | Research/Sourcing |
| CrewAI layer | Python orchestration | Active re-foundation |

Per-agent configuration lives in dotfolders: `.claude/`, `.gemini/`, `.codex/`, `.grok/`, `.deepseek/`, `.perplexity/`, `.crewai/`, etc. **These must survive the migration intact.**

### Active GitHub Actions workflows in IDAHO-VAULT

IDAHO-VAULT has 20+ active workflows — this is an active operations center, not a static archive:

| Workflow | Function |
|---|---|
| `idaho-leg-scraper.yml` | Scrapes Idaho Legislature for new bills and updates vault |
| `daily-rollover.yml` | Daily vault maintenance and file rollover |
| `linear-brief.yml`, `linear-webhook.yml`, `linear-pr-sync.yml` | Linear project management integration (requires Linear API secrets) |
| `budget-tracker-csv-export.yml` | Exports budget tracker data to CSV |
| `vault-courier.yml`, `vault-ingest.yml` | Automated vault content routing and ingestion |
| `wayback-preserve.yml`, `wayback-audit.yml` | Wayback Machine preservation for journalism sources |
| `agent-review-gate.yml` | Gate for agent-submitted PRs — requires Logan's approval |
| `auto-pr.yml`, `auto-merge.yml` | Automated PR creation and merge flows |
| `branch-cleanup.yml`, `branch-garden-report.yml` | Git branch hygiene |
| `check-portable-paths.yml` | Enforces NETWEB cross-platform path portability standard |
| `1password-secret-template.yml` | 1Password secrets integration template |

All of these workflows depend on secrets (Linear API token, 1Password credentials, GitHub token, etc.) that must be **re-authorized in LAF-US/LAF-PRIVATE** after migration.

### Why IDAHO-VAULT must be PRIVATE

IDAHO-VAULT is currently listed as a **public** repository, which is a security concern:
- It contains agent configuration, capability tier definitions, and operational protocols
- It contains personal budget data, project notes, and private journalism research
- It contains `.github/CODEOWNERS`, issue templates, and swarm coordination scripts
- `swarm.json` contains the machine-readable registry of all connectors and agents

Moving it to `LAF-US/LAF-PRIVATE` as a **private** repo is the correct and necessary action.

### Obsidian vault integrity post-migration

IDAHO-VAULT's 1,000+ Obsidian notes use `[[wikilink]]` syntax for internal links. These resolve by filename — as long as the flat file structure is preserved, all internal links will continue to work after migration to LAF-PRIVATE.

The `.obsidian/` folder (workspace config, plugins, themes) must be preserved intact. Per the vault's TRIPLEX protocol, this folder is owned by Claude (The Abhorsen) and must not be modified during migration.

---

## Suggested Improvements to Advance the Migration

Based on review of all six repositories, the following improvements are recommended before executing the migration.

### 1. Update `swarm.json` before merging IDAHO-VAULT

`swarm.json` is the machine-readable agent registry and almost certainly contains hardcoded URLs pointing to `loganfinney27/IDAHO-VAULT`, `loganfinney27/IR-Court-Tracker`, etc. These will all break after migration. **Update `swarm.json` to reflect the new `LAF-US/LAF-PRIVATE` and `LAF-US/LAF-PUBLIC` URLs before or during the migration.**

### 2. Update cross-repo governance links in public repos

`IR-Court-Tracker/AGENTS.md` and `IR-Court-Tracker/README.md` contain hardcoded links to:
- `github.com/loganfinney27/IDAHO-VAULT/blob/main/CONSTITUTION.md`
- `github.com/loganfinney27/IDAHO-VAULT/blob/main/AGENTS.md`
- `github.com/loganfinney27/IDAHO-VAULT/blob/main/!/AGENTS.md`

These must be updated to point to `github.com/LAF-US/LAF-PRIVATE/blob/main/...` after migration. Check all other repos for similar hardcoded IDAHO-VAULT links.

### 3. Migrate IDAHO-VAULT first (it is the governance backbone)

Because all other repos depend on IDAHO-VAULT for agent governance, consider migrating IDAHO-VAULT to `LAF-US/LAF-PRIVATE` **before** migrating the public repos to LAF-PUBLIC. This ensures the governance backbone is stable before downstream repos are moved.

### 4. Re-authorize all GitHub Actions secrets in LAF-PRIVATE

IDAHO-VAULT's workflows require secrets that must be re-added to `LAF-US/LAF-PRIVATE` before any workflows run:
- Linear API token (for linear-brief, linear-webhook, linear-pr-sync)
- 1Password integration credentials
- Wayback Machine API key (if applicable)
- GitHub PAT or fine-grained token for cross-repo writes

Go to **LAF-US/LAF-PRIVATE → Settings → Secrets and variables → Actions** and add each secret before enabling the workflows.

### 5. Re-wire IR-Court-Tracker's daily workflow in LAF-PUBLIC

`IR-Court-Tracker`'s `python-run.yml` runs daily and commits `output.csv` and `failed_urls.csv` back to the repository. After migration to LAF-PUBLIC:
- Confirm `permissions: contents: write` is still set in the workflow
- Confirm the workflow's `git commit` step references the correct repo and branch
- Verify the `GITHUB_TOKEN` for `LAF-US/LAF-PUBLIC` has write access

### 6. Resolve the `output.csv` naming collision

Both IR-Court-Tracker and PyTutorial have `output.csv`. IR-Court-Tracker's file is **actively overwritten daily** by its workflow. PyTutorial's file is empty. Decide which file takes precedence and rename the other before merging.

### 7. Clean `.DS_Store` from IDEX_Artifacts before merging

IDEX_Artifacts has a `.DS_Store` file at the repo root (a macOS filesystem artifact that should never be committed). Remove it before the flat merge and add `.DS_Store` to LAF-PUBLIC's `.gitignore`.

### 8. Create a LAF-US org-level PAT for agent automation

The swarm's cross-repo automation (auto-pr, auto-merge, vault-courier, etc.) uses `GITHUB_TOKEN`. After moving to a GitHub Organization, consider creating a fine-grained organization-level PAT or a dedicated bot account (`laf-bot`) so cross-repo workflows don't depend on personal access tokens. Document this in `swarm.json` and `CONSTITUTION.md`.

### 9. Set up branch protection and team permissions in LAF-US

Once the org exists:
- Enable **branch protection on `main`** for both LAF-PUBLIC and LAF-PRIVATE (require PR review before merge — Logan is sole approver)
- Mirror the existing CODEOWNERS pattern from `IDAHO-VAULT/.github/CODEOWNERS` into the new repos
- Apply the `SWARM` label in Linear to track the migration as an active workstream

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

### Step 4: Create the LAF-PRIVATE repository and migrate IDAHO-VAULT

> **Recommended order:** Migrate IDAHO-VAULT to LAF-PRIVATE **first**, before the public repos. It is the governance backbone — getting it stable and at its new URL early reduces the window during which cross-repo governance links are broken.

```bash
gh repo create LAF-US/LAF-PRIVATE --private --description "Private LAF projects and archives"
```

Update `swarm.json` to replace all `loganfinney27/IDAHO-VAULT` URLs with `LAF-US/LAF-PRIVATE` before merging.

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

After pushing, re-add all GitHub Actions secrets in **LAF-US/LAF-PRIVATE → Settings → Secrets and variables → Actions** (Linear API token, 1Password credentials, Wayback Machine key, GitHub PAT).

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
- [ ] Set up branch protection rules and CODEOWNERS in LAF-US org
- [ ] Create LAF-US org-level PAT or bot account for swarm automation
- [ ] **Pre-flight:** Review conflict map (README.md, output.csv, requirements.txt, .gitignore, .gitattributes, package.json, workflows, index.html, AGENTS.md)
- [ ] **Pre-flight:** Note THE-GEMSTONE custom domain (`thegemstone.org`) and DNS settings
- [ ] **Pre-flight:** Clean `.DS_Store` from IDEX_Artifacts; add to LAF-PUBLIC `.gitignore`
- [ ] **Pre-flight:** Update `swarm.json` with new `LAF-US` repo URLs
- [ ] **Pre-flight:** Resolve `output.csv` naming collision (IR-Court-Tracker vs PyTutorial)
- [ ] Create `LAF-PRIVATE` repo (private)
- [ ] Merge `IDAHO-VAULT` → `LAF-PRIVATE` (flat) — **migrate first as the governance backbone**
- [ ] Re-authorize all IDAHO-VAULT GitHub Actions secrets in LAF-PRIVATE (Linear, 1Password, Wayback, etc.)
- [ ] Verify Obsidian vault integrity: open LAF-PRIVATE in Obsidian and check wikilink graph
- [ ] Create `LAF-PUBLIC` repo (public)
- [ ] Merge `PyTutorial` → `LAF-PUBLIC` (flat)
- [ ] Merge `THE-GEMSTONE` (Quartz fork) → `LAF-PUBLIC` (flat) — resolve build-system file conflicts
- [ ] Merge `IR-Court-Tracker` → `LAF-PUBLIC` (flat)
- [ ] Merge `IDEX_Artifacts` → `LAF-PUBLIC` (flat)
- [ ] Merge `loganfinney27.github.io` → `LAF-PUBLIC` (flat)
- [ ] Enable GitHub Pages on LAF-PUBLIC (Source: GitHub Actions)
- [ ] Re-configure custom domain `thegemstone.org` on LAF-PUBLIC and update DNS
- [ ] Verify Quartz site builds and deploys from LAF-PUBLIC
- [ ] Re-wire IR-Court-Tracker daily workflow in LAF-PUBLIC and verify commits run
- [ ] Update cross-repo governance links in IR-Court-Tracker (AGENTS.md, README.md) to point to LAF-PRIVATE
- [ ] Archive all original repositories
- [ ] Update any external links/references

---

## References

- [GitHub Docs: About Organizations](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)
- [GitHub Docs: Archiving a repository](https://docs.github.com/en/repositories/archiving-a-github-repository/archiving-repositories)
- [GitHub Docs: git subtree](https://git-scm.com/book/en/v2/Git-Tools-Subtree-Merging)
- [Community Discussion: Organizing repos](https://github.com/orgs/community/discussions/143759)
