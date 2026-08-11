---
name: zcee-page-maintenance
description: Use when the user wants to edit, update, re-order, or remove content on pages/subsections, or delete obsolete PDFs, assets, or pages in the WSC-ZCEE-Material GitHub site — handles editing markdown text/headings, removing obsolete PDF files, updating references in index.md / _config.yml, cleaning up orphan files, committing on a feature branch, and opening a Pull Request.
---

# WSC ZCEE Material — Page Maintenance & Cleanup Skill

Follow every step in order. Do not skip steps. Do not run any git command without explicit user confirmation first.

---

## Step 1 — Identify the Maintenance Task

Ask the user (or infer from their message) the following:

1. **What type of maintenance is required?** Classify into one or more of the scenarios below:
   - **Scenario A — Page or Subsection Text/Structure Editing**: Modifying text, headings, layout, back-links, or descriptions on an existing page without removing files.
   - **Scenario B — Removing a Subsection / Embed Block**: Deleting a specific section or PDF embed block from a page (e.g. removing a retired lab section from a multi-lab page).
   - **Scenario C — Deleting Obsolete PDFs / Assets**: Removing outdated or deprecated PDF files from `docs/pdfs/` that are no longer used.
   - **Scenario D — Deleting an Entire Page / Sub-page**: Deleting a `.md` page file, its associated PDFs, and updating parent pages, `index.md`, and `_config.yml`.
   - **Scenario E — Orphan PDF Cleanup**: Auditing and removing unused PDF files in `docs/pdfs/` that have no corresponding markdown references.

2. **Which target file(s) or PDF(s) are involved?**
   - Markdown file path(s) (e.g., `docs/labs/OAS2/CICS.md`, `docs/cobol-samples.md`)
   - PDF filename(s) (e.g., `Old_Lab_v1.pdf`)

Use `ask_followup_question` if anything is unclear or incomplete.

---

## Step 2 — Locate Target Files & Audit Dependencies

Use `grep` or `read_file` to search for references before making any changes.

### Repository Map Quick-Reference

| Section | Markdown File(s) | PDF Folder |
|---|---|---|
| Case Studies & Educational Material | `docs/case-studies.md` | `docs/pdfs/case-studies/` |
| API Requester Labs | `docs/labs/api-requester.md`, `docs/labs/Requester/*.md` | `docs/pdfs/labs/api-requester/` |
| OpenAPI 2 Labs | `docs/labs/openapi2.md`, `docs/labs/OAS2/*.md` | `docs/pdfs/labs/OAS2/` |
| OpenAPI 3 Labs | `docs/labs/openapi3.md`, `docs/labs/OAS3/*.md` | `docs/pdfs/labs/OAS3/` |
| Liberty Labs | `docs/labs/liberty.md`, `docs/labs/liberty/*.md` | `docs/pdfs/labs/liberty/` |
| Workshops | `docs/workshops/index.md`, `docs/workshops/liberty.md`, `docs/workshops/zosconnect.md` | `docs/pdfs/workshops/Liberty/`, `docs/pdfs/workshops/ZCEE/` |
| ZCEE Container Material | `docs/zcee-container.md` | `docs/pdfs/zcee-container/` |
| Topics Guides | `docs/topics-guide.md`, `docs/archive-topics-guide.md` | `docs/pdfs/topics-guide/` |
| Sample Code Pages | `docs/cntl-samples.md`, `docs/jcl-samples.md`, `docs/xml-samples.md`, `docs/cobol-samples.md` | `docs/pdfs/*-samples/` |

### Dependency Audit Rules
- If **deleting or updating a PDF file**: Use `grep` to search across all `.md` files for the PDF filename to ensure no other pages reference it:
  ```
  pattern: "[Filename].pdf"
  ```
- If **deleting a markdown page**: Check for references in:
  1. `index.md` (Homepage section list)
  2. `_config.yml` (`nav:` sidebar block)
  3. Parent category page (e.g. `docs/labs/liberty.md`)
  4. Other markdown pages linking to it via relative links

---

## Step 3 — Execute the Maintenance Scenario

### Scenario A — Page or Subsection Text/Structure Editing

1. Use `read_file` on the target markdown file.
2. Apply modifications using `search_and_replace` or `apply_diff`.
3. Ensure formatting, headings, and HTML elements (`iframe`, `a.ibm-btn`, `div.doc-meta`) remain valid and compliant with site conventions.

### Scenario B — Removing a Subsection / Embed Block

When removing a subsection from a page (e.g., a retired lab):
1. Use `read_file` to locate the section heading and its associated 3-element PDF block:
   ```html
   <iframe src="..."></iframe>
   <a href="..." class="ibm-btn ibm-btn-secondary">Full Screen Preview PDF</a>
   <a href="..." class="ibm-btn ibm-btn-primary">Download PDF</a>
   <div class="doc-meta">...</div>
   ```
2. Use `apply_diff` or `search_and_replace` to remove the subsection header, description, iframe, download links, and `doc-meta` block.
3. If the associated PDF file in `docs/pdfs/` is no longer needed, use `execute_command` to stage its deletion:
   ```bash
   git rm "docs/pdfs/[folder]/[Filename.pdf]"
   ```

### Scenario C — Deleting Obsolete PDFs / Assets

1. Verify via `grep` that the PDF file is not referenced in any `.md` file.
2. Use `execute_command` to remove the file from git tracking and disk:
   ```bash
   git rm "docs/pdfs/[folder]/[Obsolete_Filename.pdf]"
   ```
3. If the file was not tracked in git yet, remove it with `rm`:
   ```bash
   rm "docs/pdfs/[folder]/[Obsolete_Filename.pdf]"
   ```

### Scenario D — Deleting an Entire Page / Sub-page

1. Use `execute_command` to delete the page file:
   ```bash
   git rm "docs/[path]/[Page].md"
   ```
2. Check if the page has dedicated PDFs in `docs/pdfs/[folder]/`. If those PDFs are no longer used elsewhere, delete them:
   ```bash
   git rm "docs/pdfs/[folder]/[Filename.pdf]"
   ```
3. Remove the page link from `index.md` using `apply_diff`.
4. If listed in `_config.yml`, remove its entry from the `nav:` hierarchy using `apply_diff`.
5. If listed on a parent index page (e.g. `docs/labs/liberty.md`), remove the link and description item from the parent markdown file using `apply_diff`.

### Scenario E — Orphan PDF Cleanup

1. List all PDF files under `docs/pdfs/` using `glob`:
   `pattern: "docs/pdfs/**/*.pdf"`
2. For each PDF, check if its filename appears in any `.md` file using `grep`.
3. Present the list of orphan PDFs to the user.
4. Upon user confirmation, remove the orphan PDFs using `git rm`.

---

## Step 4 — Site Navigation & Homepage Reconciliation

Whenever pages or sections are deleted, edited, or re-ordered:

1. **`index.md` (Homepage)**:
   - If a top-level section was removed or re-ordered, update the numbered headings (`### **1. ...**`, `### **2. ...**`) so section numbers remain strictly sequential.
   - If a subsection link was removed, remove its bullet point under `#### **Subsections**`.

2. **`_config.yml`**:
   - Ensure `nav:` entries match active pages.
   - If a `nav_order` value was changed or removed, re-sequence `nav_order` numbers across remaining items if necessary.

3. **Parent Pages**:
   - Update list entries or link text on parent index pages.

---

## Step 5 — Pre-Commit Checklist

Before proceeding to any git commands, verify every item below and report the status to the user:

- [ ] All requested text/structural edits in `.md` files have been made
- [ ] Obsolete PDF files have been removed from `docs/pdfs/` (using `git rm`)
- [ ] Deleted `.md` files have been removed (using `git rm`)
- [ ] No broken PDF references or relative links remain in active `.md` files
- [ ] `index.md` section numbers and links are updated and sequential
- [ ] `_config.yml` navigation list is clean and up to date
- [ ] No unrelated files have been modified or deleted

If any item fails, resolve it before continuing.

---

## Step 6 — Confirm with the User Before Running Git Commands

Present a clear summary to the user:

```
Ready to commit the following maintenance changes:
  - Modified files: [list of modified .md / yaml files]
  - Deleted files:  [list of deleted .md / .pdf files]
  - Branch:         maintenance/[short-description]
  - Commit:         chore(content): [brief description of work]

Shall I proceed with creating the branch, committing, and pushing?
```

**Do not run git commit or push commands until the user explicitly confirms.**

---

## Step 7 — Git Workflow

Once the user confirms, execute the following steps in order.

### 7a — Create a Feature Branch

Branch naming convention: `maintenance/[short-description]` or `delete/[short-description]`

Examples:
- `maintenance/remove-retired-oas2-lab`
- `delete/obsolete-cics-pdfs`
- `maintenance/cleanup-orphan-pdfs`
- `maintenance/update-workshops-layout`

```bash
git checkout -b maintenance/[short-description]
```

### 7b — Stage Changes

```bash
git add docs/[modified-page].md
git add index.md
git add _config.yml
```

(Note: Deleted files handled via `git rm` are already staged for removal.)

### 7c — Commit

Commit message format: `chore(content): [description]` or `refactor(content): [description]`

Examples:
- `chore(content): remove obsolete OpenAPI 2 CICS lab PDF`
- `refactor(content): update section layout on Liberty workshops page`
- `chore(content): delete retired lab page and clean up nav links`

```bash
git commit -m "chore(content): [description]"
```

### 7d — Push the Branch

```bash
git push --set-upstream origin maintenance/[short-description]
```

### 7e — Open a Pull Request

After pushing, open a Pull Request. The PR description should summarize:
- What pages/sections were edited or removed
- Which PDF files were deleted
- What navigation or homepage updates were performed

---

## Step 8 — Report Completion

Tell the user:
- The branch name and commit hash
- Summary of deleted or modified files
- The PR URL (once created)
- A reminder that GitHub Pages will rebuild automatically once the PR is merged into `main`

---

## Trigger Phrases (examples that activate this skill)

- "Delete the obsolete PDF for the OAS2 CICS lab"
- "Remove the retired section from the Liberty workshops page"
- "Clean up orphan PDFs in docs/pdfs/"
- "Delete the old sub-page for z/OS Connect and update navigation"
- "Edit the layout and text on docs/case-studies.md"
- "Remove unused PDF files from the repository"
- "Update index.md and _config.yml to remove a deleted section"
