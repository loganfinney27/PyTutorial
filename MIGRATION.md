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

### LAF-PUBLIC

```
LAF-PUBLIC/
├── README.md
├── github-pages/          ← loganfinney27.github.io
├── the-gemstone/          ← THE-GEMSTONE
├── ir-court-tracker/      ← IR-Court-Tracker
├── idex-artifacts/        ← IDEX_Artifacts
└── py-tutorial/           ← PyTutorial (this repo)
```

### LAF-PRIVATE

```
LAF-PRIVATE/
├── README.md
└── idaho-vault/           ← IDAHO-VAULT (Obsidian archive)
```

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

### Step 3: Migrate each public repo into LAF-PUBLIC as a subdirectory

Use `git subtree` to preserve commit history for each project:

```bash
# Clone the new empty LAF-PUBLIC repo
git clone https://github.com/LAF-US/LAF-PUBLIC.git
cd LAF-PUBLIC

# Add each existing repo as a remote and merge it into a subdirectory
git remote add py-tutorial https://github.com/loganfinney27/PyTutorial.git
git fetch py-tutorial
git merge --allow-unrelated-histories py-tutorial/main  # optional: squash first
# Move only the files that belong to this project (adjust list as needed):
mkdir -p py-tutorial
git mv main.ipynb input.txt output.csv requirements.txt MIGRATION.md py-tutorial/ 2>/dev/null || true
git commit -m "chore: migrate PyTutorial into py-tutorial/"

# Repeat for each remaining repo, creating a dedicated subdirectory each time:
#   git remote add the-gemstone https://github.com/loganfinney27/THE-GEMSTONE.git
#   git fetch the-gemstone && git merge --allow-unrelated-histories the-gemstone/main
#   mkdir -p the-gemstone && git mv <files…> the-gemstone/
#   git commit -m "chore: migrate THE-GEMSTONE into the-gemstone/"
# (and so on for ir-court-tracker, idex-artifacts, github-pages)
```

> **Tip:** If preserving full history is not important, you can simply copy the files into subdirectories without the remote merge step.

### Step 4: Create the LAF-PRIVATE repository

```bash
gh repo create LAF-US/LAF-PRIVATE --private --description "Private LAF projects and archives"
```

Then migrate IDAHO-VAULT into `idaho-vault/` using the same `git subtree` approach.

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
- [ ] Create `LAF-PUBLIC` repo (public)
- [ ] Migrate `loganfinney27.github.io` → `LAF-PUBLIC/github-pages/`
- [ ] Migrate `THE-GEMSTONE` → `LAF-PUBLIC/the-gemstone/`
- [ ] Migrate `IR-Court-Tracker` → `LAF-PUBLIC/ir-court-tracker/`
- [ ] Migrate `IDEX_Artifacts` → `LAF-PUBLIC/idex-artifacts/`
- [ ] Migrate `PyTutorial` → `LAF-PUBLIC/py-tutorial/`
- [ ] Create `LAF-PRIVATE` repo (private)
- [ ] Migrate `IDAHO-VAULT` → `LAF-PRIVATE/idaho-vault/`
- [ ] Archive all original repositories
- [ ] Update any external links/references

---

## References

- [GitHub Docs: About Organizations](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/about-organizations)
- [GitHub Docs: Archiving a repository](https://docs.github.com/en/repositories/archiving-a-github-repository/archiving-repositories)
- [GitHub Docs: git subtree](https://git-scm.com/book/en/v2/Git-Tools-Subtree-Merging)
- [Community Discussion: Organizing repos](https://github.com/orgs/community/discussions/143759)
