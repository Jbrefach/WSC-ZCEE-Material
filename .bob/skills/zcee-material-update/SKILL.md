---
name: zcee-material-update
description: Use when the user wants to update a PDF or page in the WSC-ZCEE-Material GitHub Pages repository — handles copying PDFs into the correct docs/pdfs/ folder, updating iframe and anchor links in the matching markdown page, refreshing the doc-meta date, committing on a feature branch, and opening a Pull Request.
---

# WSC ZCEE Material — Page & PDF Update Skill

Follow every step in order. Do not skip steps. Do not run any git command without explicit user confirmation first.

---

## Step 1 — Identify What Is Being Updated

Ask the user (or infer from their message) the following:

1. **Which page / section** is being updated? (e.g. "OpenAPI 2 CICS lab", "case studies SAF security", "Liberty workshop")
2. **What kind of update is this?** Classify into one of the three scenarios below:
   - **Scenario A** — The new PDF file is already placed inside the repo under `docs/pdfs/`
   - **Scenario B** — The user has a PDF at a local path outside the repo and wants it copied in
   - **Scenario C** — Only the metadata date needs updating; no PDF file change

3. **Is the PDF filename changing?** (old filename → new filename, or same filename as before)

Use `ask_followup_question` if anything is unclear.

---

## Step 2 — Locate the Target Files Using the Repository Map

Use the table below to find the correct markdown file and PDF storage folder for the section being updated.

| Section | Markdown File(s) | PDF Folder |
|---|---|---|
| Case Studies & Educational Material | `docs/case-studies.md` | `docs/pdfs/case-studies/` |
| API Requester Labs (index) | `docs/labs/api-requester.md` | `docs/pdfs/labs/api-requester/` |
| API Requester Labs (individual) | `docs/labs/Requester/*.md` | `docs/pdfs/labs/api-requester/` |
| OpenAPI 2 Labs (index) | `docs/labs/openapi2.md` | `docs/pdfs/labs/OAS2/` |
| OpenAPI 2 – CICS | `docs/labs/OAS2/CICS.md` | `docs/pdfs/labs/OAS2/` |
| OpenAPI 2 – Db2 | `docs/labs/OAS2/Db2.md` | `docs/pdfs/labs/OAS2/` |
| OpenAPI 2 – IMS | `docs/labs/OAS2/IMS.md` | `docs/pdfs/labs/OAS2/` |
| OpenAPI 2 – MQ | `docs/labs/OAS2/MQ.md` | `docs/pdfs/labs/OAS2/` |
| OpenAPI 3 Labs (index) | `docs/labs/openapi3.md` | `docs/pdfs/labs/OAS3/` |
| OpenAPI 3 Labs (individual) | `docs/labs/OAS3/*.md` | `docs/pdfs/labs/OAS3/` |
| Liberty Labs (index) | `docs/labs/liberty.md` | `docs/pdfs/labs/liberty/` |
| Liberty Labs (individual) | `docs/labs/liberty/*.md` | `docs/pdfs/labs/liberty/` |
| Workshops (index) | `docs/workshops/index.md` | — |
| Liberty Workshops | `docs/workshops/liberty.md` | `docs/pdfs/workshops/Liberty/` |
| z/OS Connect Workshops | `docs/workshops/zosconnect.md` | `docs/pdfs/workshops/ZCEE/` |
| ZCEE Container Material | `docs/zcee-container.md` | `docs/pdfs/zcee-container/` |
| ZCEE Topics Guide | `docs/topics-guide.md` | `docs/pdfs/topics-guide/` |
| Archive Topics Guide | `docs/archive-topics-guide.md` | `docs/pdfs/topics-guide/` |
| ZCEE30.CNTL Samples | `docs/cntl-samples.md` | `docs/pdfs/cntl-samples/` |
| Admin JCL Samples | `docs/jcl-samples.md` | `docs/pdfs/jcl-samples/` |
| XML Samples | `docs/xml-samples.md` | `docs/pdfs/xml-samples/` |
| COBOL Samples | `docs/cobol-samples.md` | `docs/pdfs/cobol-samples/` |

### Relative Path Rules for iframe / anchor src

The `src` and `href` values in a markdown file depend on how deep the file is nested:

| Markdown file location | PDF path prefix to use |
|---|---|
| `docs/*.md` (top-level, e.g. `case-studies.md`) | `pdfs/[folder]/[filename]` |
| `docs/labs/*.md` (one level in, e.g. `openapi2.md`) | `../pdfs/[folder]/[filename]` |
| `docs/labs/OAS2/*.md` (two levels in, e.g. `CICS.md`) | `../../pdfs/[folder]/[filename]` |
| `docs/workshops/*.md` | `../pdfs/[folder]/[filename]` |

Use `read_file` on the target markdown file to verify the existing prefix pattern before making any changes.

---

## Step 3 — Execute the Correct Scenario

### Scenario A — PDF already placed in the repo

1. Use `read_file` to confirm the PDF file exists at the expected path under `docs/pdfs/`.
2. Note the exact filename (including any spaces).
3. Proceed to **Step 4** (markdown update).

### Scenario B — PDF is at a local path outside the repo

1. Ask the user for the full local path to the new PDF file if not already given.
2. Use `execute_command` to copy the file into the correct `docs/pdfs/[folder]/` directory:
   ```bash
   cp "/path/to/source/New Filename.pdf" "docs/pdfs/[folder]/New Filename.pdf"
   ```
3. Preserve the exact filename (spaces and all) as provided by the user.
4. Confirm the copy succeeded before proceeding to **Step 4**.

### Scenario C — Metadata update only (no PDF change)

1. Use `read_file` on the target markdown file to locate the `<div class="doc-meta">` block(s).
2. Identify the correct block for the section being updated (match by surrounding heading or PDF filename).
3. Update **only the date value** inside the block — do not change the label text (`Last updated:` or `Lab Version Date:`).
4. Skip to **Step 5** (skip git staging of any PDF file).

---

## Step 4 — Update the Markdown File

Every PDF entry in a markdown file has exactly **three HTML elements** that reference the PDF path. All three must be updated together whenever the PDF filename changes.

### The Three-Element Pattern

```markdown
<iframe
  src="[PATH_TO_PDF]"
  width="100%"
  height="600px"
  style="border:1px solid #ccc">
</iframe>

<a href="[PATH_TO_PDF]"
   target="_blank"
   class="ibm-btn ibm-btn-secondary">
  Full Screen Preview PDF
</a>

<a href="[PATH_TO_PDF]"
   download
   class="ibm-btn ibm-btn-primary">
  Download [label text]
</a>
```

### Update Rules

1. Use `read_file` on the markdown file first to see the current state.
2. If the PDF **filename is unchanged**, only the `doc-meta` date needs updating (jump to rule 5).
3. If the PDF **filename changed**, use `search_and_replace` or `apply_diff` to replace **all three** occurrences of the old filename with the new filename in one operation.
4. **Preserve exact spacing and indentation** — do not reformat surrounding lines.
5. Locate the `<div class="doc-meta">` block that belongs to this PDF section and update the date value to the new date provided by the user. Common formats used in this repo: `June 2026`, `Apr 5, 2024`, `2025`, `October 3, 2025` — match the format the user specifies.

### Multi-PDF Pages

Some pages (e.g. `docs/labs/OAS2/CICS.md`) contain two or more PDF sections. Each section has its own three-element pattern and its own `doc-meta` block.

- Identify the correct section by matching the **PDF filename** or the **heading** above the iframe.
- Only modify the elements belonging to the target section. Leave all other sections untouched.

### Filename Spaces Warning

PDF filenames in this repo regularly contain spaces (e.g. `Developing RESTful APIs for a CICS Channel program.pdf`). Always preserve these exactly — do not encode them, do not remove spaces, do not alter capitalisation.

---

## Step 5 — Pre-Commit Checklist

Before proceeding to any git commands, verify every item below. Report the status of each to the user:

- [ ] PDF file exists at the correct path in `docs/pdfs/[folder]/`
- [ ] `<iframe src="…">` points to the correct (new) filename
- [ ] `<a href="…" target="_blank">` (preview link) points to the correct (new) filename
- [ ] `<a href="…" download>` (download link) points to the correct (new) filename
- [ ] `<div class="doc-meta">` date has been updated to the value provided by the user
- [ ] No other files have been accidentally modified
- [ ] If the PDF filename changed: old PDF file has been removed (use `execute_command`: `git rm docs/pdfs/[folder]/Old Name.pdf`)

If any item fails, fix it before continuing.

---

## Step 6 — Confirm with the User Before Running Git Commands

Present a clear summary to the user:

```
Ready to commit the following changes:
  - PDF: docs/pdfs/[folder]/[filename]
  - Page: [markdown file path]
  - Branch: update/[category]-[short-description]
  - Commit: feat(content): update [PDF name] in [category]

Shall I proceed with creating the branch, committing, and pushing?
```

**Do not run any git command until the user explicitly confirms.**

---

## Step 7 — Git Workflow

Once the user confirms, execute the following steps in order.

### 7a — Create a Feature Branch

Branch naming convention: `update/[category]-[short-description]`

Examples:
- `update/oas2-cics-lab`
- `update/case-studies-saf-security`
- `update/liberty-workshops`
- `update/topics-guide-date`

```bash
git checkout -b update/[category]-[short-description]
```

### 7b — Stage the Changed Files

Stage the markdown file and the PDF file (if it changed):

```bash
git add docs/[path/to/page].md
git add "docs/pdfs/[folder]/[New Filename.pdf]"
```

If an old PDF was removed:
```bash
git rm "docs/pdfs/[folder]/Old Filename.pdf"
```

### 7c — Commit

Commit message format: `feat(content): update [PDF name] in [category]`

Examples:
- `feat(content): update CICS Channel lab PDF in OAS2`
- `feat(content): update doc-meta date for SAF Security case study`
- `feat(content): replace Liberty workshop PDF in workshops/Liberty`

```bash
git commit -m "feat(content): update [PDF name] in [category]"
```

### 7d — Push the Branch

```bash
git push --set-upstream origin update/[category]-[short-description]
```

### 7e — Open a Pull Request

After pushing, use the built-in `create_pr_workflow` to open a Pull Request. The PR description should include:
- Which page was updated
- Which PDF(s) changed (old name → new name if filename changed)
- The new doc-meta date set

---

## Step 8 — Report Completion

Tell the user:
- The branch name and commit hash
- The PR URL (once created)
- A reminder that GitHub Pages will rebuild automatically once the PR is merged into `main`

---

## Trigger Phrases (examples that activate this skill)

- "Update the OpenAPI 2 CICS lab PDF"
- "Replace the SAF Security case study with the new version"
- "The Liberty workshop PDF has been updated, can you push it"
- "Update the last updated date on the topics guide page"
- "I've put a new PDF in docs/pdfs/workshops/ZCEE, can you update the page and open a PR"
- "Copy this PDF into the repo and update the download link"
- "Refresh the doc-meta date on the Db2 lab page"
