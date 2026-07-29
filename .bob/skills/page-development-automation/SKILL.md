---
name: page-development-automation
description: Use when the user wants to create a new page, section, or sub-page in the WSC-ZCEE-Material GitHub Pages site — handles choosing the correct location, creating the markdown file with proper frontmatter, creating the PDF collateral folder, adding the entry to index.md and _config.yml nav, then committing on a feature branch and opening a Pull Request.
---

# WSC ZCEE Material — New Page Development Automation

Follow every step in order. Do not skip steps. Do not run any git command without explicit user confirmation first.

---

## Step 1 — Gather Page Requirements

Ask the user (or infer from their message) the following. Use `ask_followup_question` for anything unclear.

1. **Page title** — what will the `title:` frontmatter and `<h1>` heading say?
2. **Short description** — one or two sentences describing what the page contains (used in the homepage `index.md` entry).
3. **Page type** — classify into one of the four types below:
   - **Type A — Top-level page**: stands alone in the sidebar (e.g. `docs/cobol-samples.md`). Has its own `nav_order`.
   - **Type B — Category index page**: acts as a parent for a group of sub-pages (e.g. `docs/labs/liberty.md`). Has a `parent:` and lists its children.
   - **Type C — Sub-page (leaf)**: lives under a parent category and contains the actual PDF content (e.g. `docs/labs/liberty/Config.md`).
   - **Type D — New category with both an index and sub-pages**: a brand-new top-level group that needs both a category index and at least one sub-page created together.
4. **Parent section** (Types B, C, D) — which existing category does this page fall under? (e.g. "Labs", "Workshops", "a new group called X")
5. **Desired URL slug** — the folder/filename part of the URL (e.g. `liberty`, `Config`, `my-new-section`). If not provided, derive one from the title using lowercase-with-hyphens.
6. **Initial PDFs** — list of PDF filenames the user wants to embed on this page (can be zero). For each PDF: filename, whether it is already in the repo or at a local path.
7. **Nav order** (Type A and D only) — where in the sidebar should this appear? List the current `nav_order` values from `_config.yml` so the user can choose.
8. **Date** — what `doc-meta` date should be stamped on any PDF sections? Default to today's date if not specified.

---

## Step 2 — Determine File Locations

Using the answers from Step 1, resolve every path that will be created or modified. Present this plan to the user for confirmation before writing anything.

### Location Rules

| Page Type | Markdown file path | PDF folder | `_config.yml` nav change |
|---|---|---|---|
| Type A — Top-level | `docs/[slug].md` | `docs/pdfs/[slug]/` | Add entry under `nav:` with new `nav_order` |
| Type B — Category index | `docs/[parent-slug]/[slug].md` | `docs/pdfs/[parent-slug]/[slug]/` | Add child entry under existing parent in `nav:` |
| Type C — Sub-page (leaf) | `docs/[parent-folder]/[Slug].md` | _(inherits parent's PDF folder)_ | No nav change needed (parent already listed) |
| Type D — New category | `docs/[slug]/index.md` + `docs/[slug]/[child-slug].md` | `docs/pdfs/[slug]/` | Add new top-level nav entry + children |

### Relative Path Rules for iframe / anchor `src`

The prefix used in `src` and `href` depends on nesting depth of the markdown file:

| Markdown file depth | PDF path prefix |
|---|---|
| `docs/*.md` (top-level) | `pdfs/[folder]/[filename]` |
| `docs/[section]/*.md` (one level in) | `../pdfs/[folder]/[filename]` |
| `docs/[section]/[sub]/*.md` (two levels in) | `../../pdfs/[folder]/[filename]` |

### Back-link Rules

Every page uses an IBM back button as the first element after the frontmatter:

| Page type | Back link target | Link text |
|---|---|---|
| Top-level (`docs/*.md`) | `../` | `← Back to Home` |
| Category index (`docs/labs/liberty.md`) | `./` | `← Back to [Parent Title]` |
| Sub-page (`docs/labs/liberty/Config.md`) | `../[parent-slug]` | `← Back to [Parent Title]` |

---

## Step 3 — Copy Any PDFs into the Repo

For each PDF the user provided:

### PDF already in the repo (Scenario A)
1. Use `read_file` to confirm the file exists at `docs/pdfs/[folder]/[filename]`.
2. Note the exact filename (preserve all spaces and capitalisation).

### PDF at a local path outside the repo (Scenario B)
1. Ask for the full local path if not already given.
2. Use `execute_command` to copy it:
   ```bash
   cp "/full/local/path/Filename.pdf" "docs/pdfs/[folder]/Filename.pdf"
   ```
3. Confirm the copy succeeded before continuing.

### No PDFs yet (Scenario C)
1. Still create the PDF folder so it is ready for future uploads.
2. Use `execute_command`:
   ```bash
   mkdir -p docs/pdfs/[folder]
   ```

---

## Step 4 — Create the Markdown File

Use `write_file` to create the new `.md` file. Choose the correct template below based on page type.

### Template: Type A — Top-level page (with PDF)

```markdown
---
title: "[Page Title]"
nav_order: [N]
---
<a href="../" class="ibm-btn ibm-btn-secondary"> ← Back to Home </a>

# [Page Title]

[Short description of what this page contains.]

## [Section Heading]

### [PDF Document Title]
<iframe
  src="pdfs/[folder]/[Filename.pdf]"
  width="100%"
  height="600px"
  style="border:1px solid #ccc">
</iframe>

<a href="pdfs/[folder]/[Filename.pdf]"
   target="_blank"
   class="ibm-btn ibm-btn-secondary">
  Full Screen Preview PDF
</a>

<a href="pdfs/[folder]/[Filename.pdf]"
   download
   class="ibm-btn ibm-btn-primary">
  Download [Short Label] PDF
</a>

<div class="doc-meta">
  <strong>Last updated:</strong> [Date]
</div>
```

### Template: Type B — Category index page

```markdown
---
title: "[Page Title]"
parent: "[Parent Title]"
---

<a href="./" class="ibm-btn ibm-btn-secondary"> ← Back to [Parent Title]</a>

# [Page Title]

[Short description of what this section contains.]

## Available [Labs / Documents / Workshops]

- **[Sub-page Title]**  
  [One-line description]  
  [View details →]([sub-slug]/[ChildSlug])
```

_(Add more list items for each sub-page. If the index also embeds a "read me" PDF, append the three-element iframe+anchor pattern from the Type A template.)_

### Template: Type C — Sub-page (leaf) with PDF

```markdown
---
title: "[Page Title]"
parent: "[parent-slug]"
nav_order: [N]
---
<a href="../[parent-slug]" class="ibm-btn ibm-btn-secondary"> ← Back to [Parent Title]</a>

## [Page Title]

<iframe
  src="../../pdfs/[folder]/[Filename.pdf]"
  width="100%"
  height="600px"
  style="border:1px solid #ccc">
</iframe>

<a href="../../pdfs/[folder]/[Filename.pdf]"
   target="_blank"
   class="ibm-btn ibm-btn-secondary">
  Full Screen Preview PDF
</a>

<a href="../../pdfs/[folder]/[Filename.pdf]"
   download
   class="ibm-btn ibm-btn-primary">
  Download [Short Label] PDF
</a>

<div class="doc-meta">
  <strong>Lab Version Date:</strong> [Date]
</div>
```

### Template: Type D — New category index (`docs/[slug]/index.md`)

```markdown
---
title: "[Category Title]"
nav_order: [N]
---
<a href="../../" class="ibm-btn ibm-btn-secondary"> ← Back to Home </a>

# [Category Title]

[Short description of what this category contains.]
```

_(Sub-pages under this category use the Type C template.)_

---

## Step 5 — Update `index.md` (Homepage)

Open `index.md` with `read_file` to find the current list of sections. Add a new entry following the exact same pattern as the existing sections. The section number increments from the last existing entry.

### Homepage Entry Pattern

```markdown
### [**N. [Page Title]**](docs/[path])

[Short description provided by the user in Step 1.]

```

For a Type D (category with subsections), also add a `#### **Subsections**` block:

```markdown
### [**N. [Category Title]**](docs/[slug])

[Short description.]

#### **Subsections**
- [**[Sub-page Title]**](docs/[slug]/[child-slug])
```

Insert the new entry before the `## 📬 Contact` section using `apply_diff` or `search_and_replace`. Use the correct sequential number (check the last `###` entry for the current count).

---

## Step 6 — Update `_config.yml` Navigation

Open `_config.yml` with `read_file` to see the current `nav:` block.

### Rules

- **Type A / Type D**: Add a new top-level `nav:` entry. Place it at the `nav_order` position agreed in Step 1. Example:
  ```yaml
    - name: "[Nav Display Name]"
      link: "/[slug]"
      nav_order: [N]
  ```
- **Type B (new child under existing parent)**: Add a child entry under the correct parent:
  ```yaml
        - name: "[Sub-page Display Name]"
          link: "/[parent-slug]/[slug]"
  ```
- **Type C (leaf sub-page)**: No `_config.yml` change needed — the parent entry already covers it.

Use `apply_diff` to make the change. Preserve all existing indentation (2-space YAML).

---

## Step 7 — Pre-Commit Checklist

Before running any git command, verify every applicable item and report status to the user:

- [ ] New markdown file exists at the correct path
- [ ] PDF folder exists at the correct path under `docs/pdfs/`
- [ ] All PDF files referenced in the markdown file exist in the PDF folder
- [ ] `<iframe src>`, preview `<a href>`, and download `<a href>` all point to the same correct path
- [ ] `<div class="doc-meta">` date is set correctly
- [ ] Back-link (`← Back to …`) uses the correct relative path and label
- [ ] `index.md` has a new section entry with the correct number and link
- [ ] `_config.yml` nav has been updated (if required for this page type)
- [ ] No unrelated files have been accidentally modified

Fix any failing item before continuing.

---

## Step 8 — Confirm with the User Before Running Git Commands

Present a clear summary:

```
Ready to commit the following new page:
  - New page:    [markdown file path]
  - PDF folder:  docs/pdfs/[folder]/
  - PDFs added:  [list or "none yet"]
  - index.md:    new section [N] added
  - _config.yml: [nav change description or "no change"]
  - Branch:      add/[slug]-page
  - Commit:      feat(content): add [Page Title] page

Shall I proceed with creating the branch, committing, and pushing?
```

**Do not run any git command until the user explicitly confirms.**

---

## Step 9 — Git Workflow

Once the user confirms, execute in order.

### 9a — Create a Feature Branch

Branch naming: `add/[slug]-page`

Examples:
- `add/liberty-jwt-page`
- `add/cobol-samples-advanced-page`
- `add/oas3-mq-lab`

```bash
git checkout -b add/[slug]-page
```

### 9b — Stage All New and Modified Files

```bash
git add docs/[new-page-path].md
git add docs/pdfs/[folder]/          # stages the entire new PDF folder
git add index.md
git add _config.yml                  # only if modified
```

### 9c — Commit

Commit message format: `feat(content): add [Page Title] page`

```bash
git commit -m "feat(content): add [Page Title] page"
```

### 9d — Push the Branch

```bash
git push --set-upstream origin add/[slug]-page
```

### 9e — Open a Pull Request

Use the built-in `create_pr_workflow`. The PR description should include:
- The new page title and URL path
- Which section it was added under (or that it is a new top-level section)
- Which PDFs were added (if any)
- What `index.md` and `_config.yml` changes were made

---

## Step 10 — Report Completion

Tell the user:
- The branch name and commit hash
- A preview of the new page URL: `https://[org].github.io/WSC-ZCEE-Material/[path]/`
- The PR URL (once created)
- A reminder that GitHub Pages rebuilds automatically once the PR is merged into `main`
- Optionally: any follow-up work left (e.g. "PDF not yet added — drop the file into `docs/pdfs/[folder]/` and run the `zcee-material-update` skill to embed it")

---

## Repository Quick-Reference Map

| Section | Markdown | PDF folder | `_config.yml` link |
|---|---|---|---|
| Case Studies & Educational Material | `docs/case-studies.md` | `docs/pdfs/case-studies/` | `/case-studies` |
| Labs (index) | `docs/labs/index.md` | — | `/labs` |
| API Requester Labs | `docs/labs/api-requester.md` | `docs/pdfs/labs/api-requester/` | `/labs/api-requester` |
| OpenAPI 2 Labs | `docs/labs/openapi2.md` | `docs/pdfs/labs/OAS2/` | `/labs/openapi2` |
| OpenAPI 3 Labs | `docs/labs/openapi3.md` | `docs/pdfs/labs/OAS3/` | `/labs/openapi3` |
| Liberty Labs (index) | `docs/labs/liberty.md` | `docs/pdfs/labs/liberty/` | `/labs/liberty` |
| Liberty Labs (sub-pages) | `docs/labs/liberty/*.md` | `docs/pdfs/labs/liberty/` | _(no nav entry)_ |
| Workshops (index) | `docs/workshops/index.md` | — | `/workshops` |
| Liberty Workshops | `docs/workshops/liberty.md` | `docs/pdfs/workshops/Liberty/` | `/workshops/liberty` |
| z/OS Connect Workshops | `docs/workshops/zosconnect.md` | `docs/pdfs/workshops/ZCEE/` | `/workshops/zosconnect` |
| ZCEE Container Material | `docs/zcee-container.md` | `docs/pdfs/zcee-container/` | `/zcee-container` |
| ZCEE Topics Guide | `docs/topics-guide.md` | `docs/pdfs/topics-guide/` | `/topics-guide` |
| ZCEE30.CNTL Samples | `docs/cntl-samples.md` | `docs/pdfs/cntl-samples/` | `/cntl-samples` |
| Admin JCL Samples | `docs/jcl-samples.md` | `docs/pdfs/jcl-samples/` | `/jcl-samples` |
| XML Samples | `docs/xml-samples.md` | `docs/pdfs/xml-samples/` | `/xml-samples` |
| COBOL Samples | `docs/cobol-samples.md` | `docs/pdfs/cobol-samples/` | `/cobol-samples` |

Current `nav_order` values in use: 1 (Case Studies), 2 (Labs), 3 (Workshops), 4 (ZCEE Container), 5 (CNTL Samples), 6 (JCL Samples), 7 (XML Samples), 8 (Topics Guide), 9 (COBOL Samples).

---

## Trigger Phrases (examples that activate this skill)

- "Create a new page for [topic]"
- "Add a new lab page under Liberty Labs"
- "I want to add a new top-level section called [X]"
- "Build a new sub-page under Workshops for [topic]"
- "Scaffold a new page with a PDF folder and homepage link"
- "Add a new section to the site"
- "Set up a new page and push it as a PR"
