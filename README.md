# Poulex Firmware Release Playbook (Company‑wise)

**Last updated:** August 31, 2025

This guide shows a **simple, reliable** Git flow to track **company‑wise firmware releases**, where **each release can include multiple commits**. Keep it minimal and consistent.

---

## 1) Model at a glance
- `main` — your generic, stable firmware line.
- `company/<CompanyName>` — one long‑lived branch per customer.
- **Releases** are **annotated tags** placed **on the company branch** (no separate release branches).

> Why this works: it keeps history flat and traceable. Every company’s history lives on its own branch; releases are visible as tags.

---

## 2) Naming rules (copy these)
**Branches**
- `company/GreenFarm`
- `company/PoultryPlus`

**Tags (SemVer + company + artifact)**
- `GreenFarm-fw-v1.3.0`
- `PoultryPlus-fw-v0.9.2`

> Use **annotated tags** so your notes travel with the tag.

---

## 3) One‑time setup per company
```bash
git checkout main
git pull --ff-only
git checkout -b company/GreenFarm
git push -u origin company/GreenFarm
```

---

## 4) Daily development (as many commits as needed)
Work only on the company branch for that customer:
```bash
git checkout company/GreenFarm
# ...edit code...
git add .
git commit -m "feat: add PWM fan curve for GreenFarm houses"
git commit -m "fix: handle RS485 sensor dropout"
git push
```

Tips
- Make **small, focused commits** with clear messages.
- If you keep a version in code (e.g., `version.h`), only bump it **right before** tagging.

---

## 5) Cut a release (tag on the company branch)
1. Ensure tests/builds pass locally or in CI.
2. (Optional) Bump version inside code:
   ```bash
   git commit -m "chore(release): GreenFarm v1.3.0"
   ```
3. Create an **annotated tag** with release notes:
   ```bash
   git tag -a GreenFarm-fw-v1.3.0 -m "GreenFarm v1.3.0
   - PWM fan curve
   - RS485 dropout fix
   - OTA stability improvements"
   ```
4. Push branch and tags:
   ```bash
   git push origin company/GreenFarm --tags
   ```

> Attach your built firmware `.bin` to the Git hosting **Release** that points to this tag.

---

## 6) Hotfix flow
```bash
# Start from the company branch
git checkout company/GreenFarm
git pull

# Create a short-lived hotfix branch (optional but clean)
git checkout -b hotfix/GreenFarm-null-sensor

# ...fix...
git commit -m "fix: guard null sensor payload"
git push -u origin hotfix/GreenFarm-null-sensor

# Merge back to the company branch (PR or fast-forward)
git checkout company/GreenFarm
git merge --ff-only hotfix/GreenFarm-null-sensor
git push

# Tag a patch release
git tag -a GreenFarm-fw-v1.3.1 -m "Hotfix: null sensor payload"
git push origin --tags

# Tidy up (optional)
git branch -d hotfix/GreenFarm-null-sensor
git push origin --delete hotfix/GreenFarm-null-sensor
```

---

## 7) Keep company branch fresh with main (optional but healthy)
Bring generic improvements into each company’s line regularly:
```bash
git checkout company/GreenFarm
git fetch origin
git rebase origin/main      # (or: git merge origin/main)
git push --force-with-lease # only when rebasing
```

> Rebasing keeps history linear; use `--force-with-lease` to avoid overwriting others’ work by mistake.

---

## 8) Quick commands you’ll actually use
- **List all releases for a company:**
  ```bash
  git tag --list "GreenFarm-fw-v*"
  ```
- **Show release notes for a tag:**
  ```bash
  git show GreenFarm-fw-v1.3.0
  ```
- **Diff between two releases:**
  ```bash
  git diff GreenFarm-fw-v1.3.0..GreenFarm-fw-v1.3.1
  ```
- **Show commits since last release (on current branch):**
  ```bash
  git describe --tags --abbrev=0   # prints last tag
  git log <last-tag>..HEAD --oneline
  ```
- **Reset branch to a known good release (be careful):**
  ```bash
  git checkout company/GreenFarm
  git reset --hard GreenFarm-fw-v1.3.0
  git push --force-with-lease
  ```

---

## 9) Tag message template (copy/paste)
```
<CompanyName> v<version>

Highlights
- <bullet>
- <bullet>

Notes
- Required config/env changes: <none|list>
- OTA compatibility: <e.g., min bootloader v0.5>
- Rollback plan: <e.g., revert to <tag> if error rate > X%>
```

---

## 10) CI/CD pointer (optional)
- Trigger build on **tags matching** `*-fw-v*`.
- Publish artifacts as release assets (e.g., `GreenFarm-v1.3.0.bin`).
- Keep a simple compatibility matrix in the release notes if needed.

---

## 11) Safety checklist
- ✅ Work for a company **only** on its `company/<Name>` branch.
- ✅ Use **annotated tags** for every release.
- ✅ Keep commits small; write clear messages.
- ✅ Rebase/merge from `main` periodically.
- 🚫 Don’t develop on `main` for customer-specific code.
- 🚫 Don’t use lightweight tags for releases.

---

**Done.** You now have a minimal, crystal-clear flow to track company‑wise firmware releases with multiple commits per release.
