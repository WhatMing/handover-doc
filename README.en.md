# handover-doc

[简体中文](./README.md) | **English**

> A general-purpose Agent Skill that analyzes a project's **Git history and local files** (front-end, back-end, or full-stack) and generates a **handover document** — a single self-contained HTML file with an interactive mind map, ready to share, print, or export to PDF.

Built for the "the person leaves, the work stays" scenario: offboarding handovers, project transfers, and getting a successor up to speed fast.

---

## What it does

- **Git repo and local files only** — no external services, works on an intranet
- **Every claim is traceable** — work items cite commit hashes, ledger sections, or file paths; nothing unsupported is written
- **A successor can start without asking anyone** — prerequisites, install and run commands, environment variable names, and common pitfalls are all covered
- **One HTML artifact** — interactive mind map, scroll-spy table of contents, one-click code copy, A4 print / PDF export styles
- **Tailored to the project type** — auto-detects front-end / back-end / full-stack and fills only the "counterpart" roles in the contacts table

## What the output looks like

Chapters are ordered by **what a successor needs first** (get it running and dodge the traps, then read the history and the details):

1. Project overview (positioning, background, current status, repo info + contacts table + five-step quick start)
2. Mind map overview (zoom, pan, collapse branches)
3. Quick start / environment setup (prerequisites → install → run → verify → FAQ)
4. Project structure and module responsibilities
5. Functional modules in detail
6. Potential risks and caveats (rated high / medium / low)
7. Build and deployment
8. API inventory
9. Tech stack (exact versions)
10. Work completed (totals + a main-work overview; no trivial commit dumps)
11. Data model (omitted entirely for pure front-end projects)
12. Handover checklist + appendices

## Installation

### Option 1: project-level (applies to the current project only)

```bash
git clone https://github.com/WhatMing/handover-doc.git .trae/skills/handover-doc
```

### Option 2: user-level (applies to all projects)

```powershell
# Windows
git clone https://github.com/WhatMing/handover-doc.git "$env:USERPROFILE\.trae-cn\skills\handover-doc"
```

```bash
# macOS / Linux
git clone https://github.com/WhatMing/handover-doc.git ~/.trae-cn/skills/handover-doc
```

> You can also simply copy `SKILL.md` and `template.html` into a `handover-doc/` folder under your target skills directory.

## Usage

In an Agent that supports Skills (for example Trae), just say:

- `Generate a handover document for D:\path\to\project`
- `I'm leaving this role — turn the project into a handover document`
- `handover`, `project handover`, `transfer`

The skill runs a six-step workflow automatically: identify the project type → collect Git and local file information → summarize "what was done" → build the mind map → generate the HTML → verify and deliver.

**Note**: product/business owners and release personnel cannot be inferred from code. The Agent will ask you for them — please answer accurately.

## Workflow (six steps)

| Step | Description |
| --- | --- |
| 1. Identify project type | Read root-level marker files (package.json / pom.xml / go.mod…) to classify front-end / back-end / full-stack |
| 2. Gather information | Git commit details, contributor stats, branches and remotes, top-20 change hotspots; README, ledger, dependency versions, directory tree, config, routes, APIs, data models |
| 3. Summarize "what was done" | Group by feature / fix / refactor / docs, cite evidence for every item, total up the workload |
| 4. Build the mind map | Write the outline into the `MIND_DATA` tree object in `template.html` |
| 5. Generate the HTML | Produce the twelve-chapter document from the template |
| 6. Verify and deliver | Zero sensitive-value leaks, versions are measured values, both mind map rendering and fallback verified |

## Requirements

- **Access to `cdn.bootcdn.net`**: the mind map library [simple-mind-map](https://github.com/wanglin2/mind-map) 0.14.0 is loaded from BootCDN
  - The library is about 6.5 MB, so **the mind map appears only after the first download completes**
  - Async loading and a fallback are built in: while the library is not ready the text stays readable and the mind map area shows a notice — **chapters never go missing**
- **The `git` command-line tool** (used to read commit history)
- Generation environment: Trae or any Skill-capable Agent; viewing the generated document only needs a browser

## Structure

```
handover-doc/
├─ SKILL.md         # Skill definition: positioning, six-step workflow, required-contacts rules, security red lines, quality bar
├─ template.html    # Handover document template: styles + mind map + init and fallback logic
├─ README.md        # Chinese documentation
├─ README.en.md     # English documentation
└─ LICENSE
```

## Design notes (lessons from real pitfalls)

- **The mind map library must be `defer`-loaded at the end of the document**: the minified build is about 6.5 MB. Placed mid-document as a synchronous `<script src>`, it halts HTML parsing and later chapters take a long time to enter the DOM (about 53 s on an intranet in practice) — which users perceive as "missing content". The current approach is end-of-document `defer` + readiness polling + loading placeholder + failure fallback.
- **The constructor is `window.simpleMindMap.default`**: the UMD global exposes only a `default` key.
- **Use only verified view APIs**: `view.enlarge()` / `narrow()` / `fit()` / `reset()`. `expandAll` / `collapseAll` do not exist in 0.14.0 — collapse and expand use the component's built-in node dots.
- **Risk wording follows the user's account**: when a project runs normally with no outstanding issues, the chapter is titled "Potential risks and caveats" — never inflate non-blocking items into high-severity alarms that scare a successor away.
- **Zero sensitive-value leaks**: `.env.*` files are read for key names only, never values; keys, tokens, passwords, and connection strings never appear.

## Version

- **v1.3** — mind map switched to simple-mind-map (BootCDN) with `defer` async loading, a loading placeholder, and failure fallback; added the README item-by-item comparison rule and the risk-wording rule
- v1.2 — mind map replaced with the simple-mind-map component
- v1.1 — mind map switched to a local self-contained component; chapters reordered by successor priority

## License

[MIT](./LICENSE)
