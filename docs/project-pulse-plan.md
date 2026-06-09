# Mona's Project Pulse — Implementation Plan

Status: Draft plan for a small static dashboard (HTML / CSS / JSON) that the learner can open in Codespaces or a local editor preview.

This document is intended to be saved as docs/project-pulse-plan.md and used by the Orchestrator to coordinate Designer and Coder work. Agents (Orchestrator, Planner, Coder, Designer) must not stage/commit/push changes — the learner performs all git operations.

---

## Summary

Mona's Project Pulse is a compact, static dashboard that displays a set of project records (id, name, status, priority, last_updated). It consists of:

- app/index.html — static HTML that reads app/project-data.json and renders the projects as a list/cards table
- app/styles.css — responsive, accessible styles
- app/project-data.json — sample project records (JSON)
- .vscode/launch.json — strict JSON config to open index.html with cwd set to ${workspaceFolder}/app (for Codespaces/VS Code preview)

Goals:
- Minimal dependencies (static files only)
- Easy to preview locally in Codespaces / VS Code Live Preview
- Clear structure and sample data for the learner to extend

---

## Ordered Implementation Steps

1. Kickoff + file scaffolding (meta step)
2. Create data schema & sample file — app/project-data.json
3. Build skeleton HTML — app/index.html (structure + JS to load data)
4. Add styles — app/styles.css (responsive layout, status/priority color mapping)
5. Create .vscode/launch.json (strict JSON, cwd set)
6. Designer polish & accessibility pass (colors, spacing, responsive breakpoints, contrast)
7. Integration & manual validation (open preview, verify data load, accessibility checks)
8. Final checks and rollout checklist (README or short note for learner)

Each step below includes file assignments, validation points, and dependencies.

---

## File assignments (who owns what)

- Orchestrator
  - Coordinate steps and assign tasks to Designer/Coder
  - Validate final acceptance criteria
  - Do not create commits — only coordinate

- Designer (responsible for visuals & accessibility)
  - app/styles.css — color palette, responsive layout, spacing, typography
  - Provide small design deliverables: color tokens, a simple mockup image or ASCII wireframe (optional)
  - Responsible for accessibility checklist: color contrast, focus states, readable type sizes

- Coder (responsible for structure & functionality)
  - app/index.html — HTML + lightweight JavaScript to fetch and render app/project-data.json
  - app/project-data.json — sample data following provided schema
  - .vscode/launch.json — strict JSON config with cwd set to ${workspaceFolder}/app
  - Optional: small helper JS embedded into index.html (no extra files)

Files explicitly required by the spec (these must be present):
- app/index.html (Coder)
- app/styles.css (Designer + Coder collaboration)
- app/project-data.json (Coder)
- .vscode/launch.json (Coder)

(Other optional files: README.md updates, small icon assets, mockup image in docs/)

---

## Project data schema and example

Purpose: app/project-data.json must be used by index.html.

JSON schema (informational):

- type: array of project objects
- project object:
  - id: string or integer (unique)
  - name: string
  - status: string (one of "Not Started", "In Progress", "Blocked", "Complete")
  - priority: string (one of "Low", "Medium", "High")
  - last_updated: string — ISO 8601 timestamp (e.g., 2026-06-09T12:00:00Z)

Example app/project-data.json (sample content — include at least 4–6 records):

{
  "projects": [
    {
      "id": "p-001",
      "name": "Website Redesign",
      "status": "In Progress",
      "priority": "High",
      "last_updated": "2026-06-08T09:15:00Z"
    },
    {
      "id": "p-002",
      "name": "Quarterly Report",
      "status": "Not Started",
      "priority": "Medium",
      "last_updated": "2026-06-02T14:10:00Z"
    },
    {
      "id": "p-003",
      "name": "Mobile App QA",
      "status": "Blocked",
      "priority": "High",
      "last_updated": "2026-06-05T16:40:00Z"
    },
    {
      "id": "p-004",
      "name": "Marketing Email Campaign",
      "status": "Complete",
      "priority": "Low",
      "last_updated": "2026-05-30T11:00:00Z"
    }
  ]
}

Notes:
- Choosing an outer object with "projects" key is optional; index.html must match whichever structure is used. The plan assumes the above form ({"projects": [...]}) because it's explicit and easier to extend.
- last_updated uses ISO 8601 to simplify parsing/display.

---

## index.html responsibilities and example structure

Purpose: simple HTML page that loads the JSON and renders a table or grid of project cards.

Responsibilities:
- Load app/styles.css
- Load project-data.json (via fetch from a relative path)
- Render project rows/cards with columns: Name, Status, Priority, Last Updated
- Apply CSS classes for status and priority so Designer can control colors
- Minimal JS (embedded) — no frameworks
- Graceful handling of errors (missing/invalid JSON, empty list)

Important implementation note (CORS / file://):
- Fetching JSON using fetch() requires the page to be served via HTTP (relative file paths while opening a file:// URL can cause fetch to fail in modern browsers). Preferred preview method: use VS Code built-in Live Preview / Simple Browser / Codespaces preview or serve via a simple static server (Live Server extension).
- The .vscode/launch.json will help open index.html in a way that works for Codespaces. Provide instructions in README for preview if necessary.

Minimal index.html outline (for planning; **do not** commit anything automatically):

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mona's Project Pulse</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header><h1>Project Pulse</h1></header>
  <main id="main">
    <div id="status-message" role="status" aria-live="polite"></div>
    <section id="project-list" aria-label="Project list"></section>
  </main>

  <script>
    // Fetch app/project-data.json, render list, handle errors and empty dataset
    // Use relative path: './project-data.json'
    // Map statuses/priorities to CSS classes (status-in-progress, priority-high etc.)
  </script>
</body>
</html>

Key behaviors the Coder must implement:
- Fetch and parse JSON
- Validate data shape (if missing keys, show fallback text)
- Sort projects by priority and/or last_updated (optional — describe as enhancement)
- Provide a visible message if no projects exist
- Show human-readable date (local timezone) for last_updated, or relative time (optional)

---

## styles.css responsibilities and suggestions

Designer-led responsibilities:
- Provide a small color palette and token names (e.g., --color-bg, --color-text, --color-high, --color-medium, --color-low)
- Ensure WCAG contrast for text and status chips
- Provide responsive breakpoints: desktop (grid of cards), tablet, mobile (single column)
- Provide focus states for keyboard navigation
- Provide aria-friendly classes for status chips (e.g., .status-complete, .status-blocked)

Minimal tokens and class approach (example):

:root {
  --bg: #ffffff;
  --text: #1a1a1a;
  --muted: #6b7280;
  --high: #ef4444;   /* red */
  --medium: #f59e0b; /* amber */
  --low: #10b981;    /* green */
  --in-progress: #3b82f6; /* blue */
}

.status-Complete { background: var(--low); }
.status-Blocked { background: var(--high); }
.status-In\ Progress { background: var(--in-progress); }
.priority-High { border-left: 4px solid var(--high); }
.priority-Medium { border-left: 4px solid var(--medium); }
.priority-Low { border-left: 4px solid var(--low); }

Accessibility:
- Default font-size >= 16px on root
- Use rem-based spacing
- Ensure contrast for status chips and text

---

## .vscode/launch.json (strict JSON)

Requirements:
- Must be strict JSON (no comments)
- cwd must be "${workspaceFolder}/app"
- Configure to open index.html for preview

Example .vscode/launch.json content (replace "pwa-chrome" with "pwa-msedge" if preferred; this is a standard VS Code launch config):

{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Open index.html (Chrome)",
      "type": "pwa-chrome",
      "request": "launch",
      "file": "${workspaceFolder}/app/index.html",
      "cwd": "${workspaceFolder}/app"
    }
  ]
}

Notes:
- This configuration launches a browser pointing to the local file URL. If fetch() to project-data.json fails due to file:// restrictions, the learner should open the folder with Live Server or use VS Code's Simple Browser/Live Preview. Include instructions in the README.

---

## Dependencies between steps

- Step 1 (Scaffold) must be done before any file is edited (ensures directory structure).
- Step 2 (project-data.json) should come early (Coder), because index.html needs data schema and sample records to be developed/tested against.
- Step 3 (index.html) depends on Step 2 (data file) and Step 4 (styles.css) only for styling — basic HTML/JS can be implemented using placeholder styles; final polish depends on Designer.
- Step 4 (styles.css) can be worked on in parallel with Step 3 after schema is defined.
- Step 5 (.vscode/launch.json) can be created at any time after scaffolding; validation depends on viewer availability.
- Step 6 (Designer polish) depends on Steps 3 and 4 being in usable shape to refine visuals and accessibility.
- Step 7 (Integration & validation) depends on Steps 2–6.

Summary:
- Must run sequentially in part: 1 → 2 → 3 (with 4 parallelizable) → 5 → 6 → 7.

---

## Work that can run in parallel vs must be sequential

Parallelizable work:
- Designer: create color tokens and wireframe mockups while Coder creates project-data.json
- Coder: implement index.html skeleton (DOM structure + basic fetch) while Designer prepares styles
- Creating .vscode/launch.json can happen while HTML/CSS are being built

Must be sequential:
- Initial scaffolding must exist before files are placed
- index.html rendering logic must reference the data schema — data schema should be defined before final JS code
- Final accessibility polish (Designer) should be done after functional HTML is in place

---

## Designer responsibilities (detailed)

- Deliverable: app/styles.css (final), color token list, spacing scale, focus styles
- Accessibility checks:
  - Text contrast >= 4.5:1 for body text, 3:1 for large text
  - Keyboard focus styles visible and not removed
  - Use semantic markup (ensure index.html has header/main/section elements)
- Layout:
  - Card design with status chip and priority indicator
  - Mobile-first responsive design
- Provide a short design note in docs/design-notes.md (optional) describing tokens and rationale

---

## Coder responsibilities (detailed)

- Create and validate app/project-data.json with 4–8 realistic project records
- Implement app/index.html:
  - Robust fetch of ./project-data.json
  - Data validation and graceful fallback
  - Render function for projects (table or card grid)
  - Human-readable date formatting using Date APIs (no external libs)
  - Minimal inline JS inside index.html (simple, easy to inspect)
  - Minimal accessible attributes (role, aria-live for status messages)
- Create .vscode/launch.json (strict JSON) with cwd set as required
- Provide usage notes in README (how to preview and common troubleshooting)

Security note: Do not include any secret or external calls. This is fully static.

---

## Edge cases, error states, and risks

1. Fetch and file:// restrictions
   - Risk: fetch('./project-data.json') may fail if index.html opened via file:// in some browsers
   - Mitigation: Document preview suggestions (VS Code Live Preview / Live Server / Codespaces preview). Add a fallback: if fetch fails, show a descriptive error message instructing the learner to run preview server.

2. Missing or malformed data
   - Behavior: If projects field missing or wrong types, show "No projects available" and write a console warning.
   - Validation: The render code should validate each object has id/name/status/priority/last_updated before rendering; otherwise show a placeholder row with "Data missing".

3. Invalid date formats or timezone confusion
   - Risk: last_updated not ISO 8601
   - Mitigation: Parse with Date constructor and fallback to raw string; show "Unknown date" if parse fails.

4. Large dataset
   - Not expected, but if many entries exist, layout should remain performant. Use simple DOM generation; not necessary to implement virtualization for this small project.

5. Unknown status/priority string values
   - Mitigation: style defaults; map known values to CSS classes and use a default look for unrecognized values.

6. Accessibility issues (color contrast, focus)
   - Mitigation: Designer tests and manual checks using browser devtools and aXe or Lighthouse.

7. Browser compatibility
   - Keep code ES5/ES6 minimal; avoid modern unsupported APIs. Use Date and fetch; include note that older browsers might not support fetch — solution: use preview in modern Chrome/Edge.

8. File path/cwd mismatch
   - Ensure .vscode/launch.json cwd is ${workspaceFolder}/app to avoid broken relative paths.

---

## Validation expectations (per step) — tests and manual checks

Step 1: Scaffold
- Check: directories exist: app/, .vscode/
- Acceptance: empty app folder or placeholder files present

Step 2: project-data.json
- Check:
  - File exists at app/project-data.json
  - JSON parses (use editor JSON viewer)
  - Contains at least 4 projects with required fields
- Acceptance: Valid JSON matching schema above

Step 3: index.html (initial)
- Check:
  - File loads in editor as HTML
  - When previewed via server, page displays header and a status message while loading
  - Console shows no uncaught errors during fetch when served via HTTP
- Acceptance: Page fetches data and renders placeholder cards or a table (see step 7 for deeper checks)

Step 4: styles.css
- Check:
  - Styles are applied (cards have spacing, color tokens defined)
  - Responsive: shrink viewport to mobile width and ensure layout stacks
- Acceptance: Layout displays correctly on mobile and desktop widths

Step 5: .vscode/launch.json
- Check:
  - Strict JSON (validate in editor)
  - cwd is "${workspaceFolder}/app"
  - Launch config appears in the Run/Debug dropdown in VS Code
- Acceptance: Running the config opens index.html in the selected browser

Step 6: Designer Accessibility & Visuals
- Check:
  - Color contrast (Lighthouse or manual check)
  - Focus indicators visible via keyboard tabbing
  - Status chips have readable labels and colors
- Acceptance: No WCAG-blocking issues for text contrast or keyboard accessibility (basic level)

Step 7: Integration & final manual verification
- Manual checks:
  - Open preview with Live Server/Live Preview/Codespaces browser preview
  - Verify that all projects show with correct name, status, priority and last_updated human-readable
  - Test error handling: temporarily rename project-data.json and confirm page shows a helpful error
  - Test empty data: make "projects": [] and ensure a "No projects" message renders
  - Keyboard test: Tab through interactive elements (if any)
- Acceptance criteria (final):
  - Page renders project list from app/project-data.json when previewed correctly
  - CSS styles applied and responsive
  - .vscode/launch.json opens index.html with cwd as required
  - Edge cases handled as described (missing file, malformed JSON) with user-friendly messages

---

## Open questions (must be resolved before implementation or explicitly documented)

1. Data file shape: prefer "projects" array vs top-level array? (Plan assumes { "projects": [...] } — confirm)
2. Required statuses and priorities: are there additional allowed values? (Plan: Not Started, In Progress, Blocked, Complete / Low, Medium, High)
3. Preferred display mode: card grid or table? (Designers may prefer cards for mobile; confirm)
4. Date formatting: use local timezone or ISO string display? (Plan: local, human-readable like "Jun 9, 2026, 14:10")
5. Is sorting/filtering required initially? (Plan: optional enhancement; not required)
6. Which browser should be used for preview validation in Codespaces? (Recommend Chrome/Edge; confirm)
7. Should the JSON include nested fields (owner, tags)? (Not for initial MVP — can be added later)

Document answers inline in docs/ or the README as soon as decisions are made.

---

## Rollout / Preview checklist (short)

Before handing to learner / preview:

- [ ] app/ directory exists with files:
  - [ ] app/index.html
  - [ ] app/styles.css
  - [ ] app/project-data.json
- [ ] .vscode/launch.json exists and is strict JSON with cwd = "${workspaceFolder}/app"
- [ ] app/project-data.json contains at least 4 valid records as per schema
- [ ] index.html fetches ./project-data.json and renders projects; fallback messages present
- [ ] styles apply and are responsive (mobile & desktop)
- [ ] README or short preview instructions added describing "How to preview" and common issues (esp. fetch + file://)
- [ ] Designer checked color contrast and keyboard navigation
- [ ] Manual test: preview opened via Live Server/Live Preview and projects show correctly

---

## Final acceptance criteria (what "done" looks like)

- A learner can open the repository in Codespaces or local VS Code and open the Run/Debug configuration to preview app/index.html (or use Live Server).
- The page displays a list or grid of projects populated from app/project-data.json without editing code.
- Styling is responsive and accessible per the checks above.
- The launch.json is a valid strict JSON file with cwd set to ${workspaceFolder}/app and a config that opens index.html for preview.
- Basic error states are handled (missing data, malformed JSON) with clear messages.
- The plan and responsibilities are documented and assigned in docs/project-pulse-plan.md (this file).

---

## Next steps for Orchestrator

1. Confirm answers to the Open Questions and record decisions in this plan.
2. Assign the Coder and Designer tasks with the file ownership above.
3. Use the Ordered Implementation Steps and the Rollout checklist to track progress.
4. Validate final acceptance criteria before marking as complete.

---

End of plan.