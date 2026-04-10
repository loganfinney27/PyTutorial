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

Merge each existing repo directly into the root of `LAF-PUBLIC`, preserving commit history:

```bash
# Clone the new empty LAF-PUBLIC repo
git clone https://github.com/LAF-US/LAF-PUBLIC.git
cd LAF-PUBLIC

# Merge each existing repo directly at root level (no subdirectory)
git remote add py-tutorial https://github.com/loganfinney27/PyTutorial.git
git fetch py-tutorial
git merge --allow-unrelated-histories py-tutorial/main
# Resolve any filename conflicts, then:
git commit -m "chore: merge PyTutorial into LAF-PUBLIC"

git remote add the-gemstone https://github.com/loganfinney27/THE-GEMSTONE.git
git fetch the-gemstone
git merge --allow-unrelated-histories the-gemstone/main
git commit -m "chore: merge THE-GEMSTONE into LAF-PUBLIC"

git remote add ir-court-tracker https://github.com/loganfinney27/IR-Court-Tracker.git
git fetch ir-court-tracker
git merge --allow-unrelated-histories ir-court-tracker/main
git commit -m "chore: merge IR-Court-Tracker into LAF-PUBLIC"

git remote add idex-artifacts https://github.com/loganfinney27/IDEX_Artifacts.git
git fetch idex-artifacts
git merge --allow-unrelated-histories idex-artifacts/main
git commit -m "chore: merge IDEX_Artifacts into LAF-PUBLIC"

git remote add github-pages https://github.com/loganfinney27/loganfinney27.github.io.git
git fetch github-pages
git merge --allow-unrelated-histories github-pages/main
git commit -m "chore: merge loganfinney27.github.io into LAF-PUBLIC"

git push origin main
```

> **Conflict resolution:** If two repos have a file with the same name (e.g., `README.md`), Git will flag a merge conflict. Manually combine the content, `git add` the resolved file, then commit before moving to the next repo.

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
- [ ] Create `LAF-PUBLIC` repo (public)
- [ ] Merge `loganfinney27.github.io` → `LAF-PUBLIC` (flat)
- [ ] Merge `THE-GEMSTONE` → `LAF-PUBLIC` (flat)
- [ ] Merge `IR-Court-Tracker` → `LAF-PUBLIC` (flat)
- [ ] Merge `IDEX_Artifacts` → `LAF-PUBLIC` (flat)
- [ ] Merge `PyTutorial` → `LAF-PUBLIC` (flat)
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
