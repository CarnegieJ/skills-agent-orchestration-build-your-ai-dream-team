# Project Pulse Dashboard — Implementation Plan

## 1. Summary

Project Pulse is a lightweight, browser-based static dashboard that gives Mona's team an at-a-glance view of every active project's health. It is built from three files — `app/index.html`, `app/styles.css`, and `app/project-data.json` — plus a VS Code launch configuration at `.vscode/launch.json`. The HTML loads project data from JSON and renders one card per project, each showing the project name, owner, current status (On Track / At Risk / Blocked), priority (High / Medium / Low), most recent activity, and a short description. The Designer agent is solely responsible for `app/styles.css`; the Coder agent owns `app/index.html`, `app/project-data.json`, and `.vscode/launch.json`. The two work in separate files, so their phases can run in parallel once the shared contract (CSS class names and JSON schema) is established in Phase 0.

---

## 2. Implementation Steps

**Phase 0 — Establish shared contracts (sequential, Planner output consumed by both agents)**

1. Define the JSON schema: top-level `projects` array; each entry has `name`, `owner`, `status`, `recentActivity`, `priority`, and `description`.
2. Define the mandatory CSS class hooks: `.dashboard`, `.project-card`, `.status-badge`, `.priority`. These are the only names Coder may hard-code in HTML class attributes.

**Phase 1 — Parallel implementation (Designer and Coder work simultaneously, no overlapping files)**

3. **[Designer]** Author `app/styles.css` — full visual design for the dashboard using the class hooks from Phase 0.
4. **[Coder]** Author `app/project-data.json` — 6 realistic sample projects using the schema from Phase 0.
5. **[Coder]** Author `app/index.html` — semantic HTML structure that references `styles.css`, fetches/references `project-data.json`, and renders project cards using the CSS hook class names from Phase 0.
6. **[Coder]** Author `.vscode/launch.json` — launch configuration named **Run Project Pulse Dashboard** that opens `app/index.html` with `cwd` set to `${workspaceFolder}/app`.

**Phase 2 — Integration validation (sequential, after all Phase 1 files exist)**

7. Open `app/index.html` in a browser (via `.vscode/launch.json` or Live Server) and confirm all cards render with correct status badges, priority labels, and owner lines.
8. Confirm `app/styles.css` rules apply: inspect that `.project-card` has `border-radius` and `box-shadow`; confirm `.status-badge` and `.priority` are visually distinct.
9. Confirm `app/project-data.json` is valid JSON (`JSON.parse` or a linter), contains a top-level `projects` array, and every entry has all required fields.
10. Confirm `.vscode/launch.json` is valid strict JSON (no comments), contains a configuration named `Run Project Pulse Dashboard`, references `index.html`, and sets `cwd` to `${workspaceFolder}/app`.

---

## 3. File Assignments

| File | Owner Agent | Deliverable |
|---|---|---|
| `app/styles.css` | **Designer** | Complete CSS — layout, card design, status badge colours, priority treatment, responsive breakpoints, accessibility |
| `app/index.html` | **Coder** | Semantic HTML page — `<head>` linking `styles.css`, `<body>` with `.dashboard` grid, JavaScript `fetch` of `project-data.json`, dynamic card rendering using class hooks |
| `app/project-data.json` | **Coder** | Valid JSON with top-level `projects` array containing 6 sample entries — each with `name`, `owner`, `status`, `recentActivity`, `priority`, `description` |
| `.vscode/launch.json` | **Coder** | Strict JSON launch configuration — `"Run Project Pulse Dashboard"`, opens `index.html`, `cwd: ${workspaceFolder}/app` |

> **No other existing files** (`docs/*`, `README.md`, `scripts/*`) are touched by either agent. The `app/` directory is currently empty — all new files are net-new.

---

## 4. Dependencies

```
Phase 0 (contracts established)
    └── Phase 1 (all four files authored in parallel)
            └── Phase 2 (integration validation)
```

| Dependency | Reason |
|---|---|
| Phase 0 → Phase 1 | Designer needs the CSS hook names before writing `styles.css`; Coder needs the JSON schema before writing `project-data.json` and the HTML rendering loop |
| `app/project-data.json` → `app/index.html` (soft) | The HTML fetch URL and field names must match the schema; Coder can write both in the same phase since the same agent owns both |
| Phase 1 fully complete → Phase 2 | Validation requires all four files to exist before the integrated result can be checked |
| `.vscode/launch.json` → Phase 2 validation step 10 | The launch config can only be tested after it is written |

---

## 5. Parallel Work Decisions

| Parallel pair | Files | Why safe |
|---|---|---|
| Step 3 (Designer, `styles.css`) ‖ Steps 4–6 (Coder, `project-data.json` + `index.html` + `launch.json`) | Zero file overlap | Designer touches only `app/styles.css`; Coder touches only `app/project-data.json`, `app/index.html`, `.vscode/launch.json` |

The Orchestrator can fire Designer and Coder simultaneously after Phase 0 contracts are confirmed. No merge conflict is possible because the file sets are fully disjoint.

---

## 6. Sequential Work Decisions

| Sequential constraint | Why |
|---|---|
| Phase 0 must complete before Phase 1 starts | CSS hook names are embedded in HTML `class` attributes and CSS selectors; JSON field names are referenced in the HTML rendering loop. Mismatches discovered late are expensive to fix. |
| Phase 2 must run after Phase 1 | Validation requires all four files to exist and be syntactically correct. |
| Steps 7–10 within Phase 2 should run in order | Each check builds on the prior one (open browser → confirm CSS → confirm JSON → confirm launch config). |
| `.vscode/launch.json` is created after `index.html` | The launch config references the HTML file by relative path; the Coder should confirm the file exists before declaring the config complete, even though authoring order is the same phase. |

---

## 7. Designer Responsibilities

The Designer agent is assigned **only `app/styles.css`**. No HTML or JSON files may be modified.

### CSS Architecture

- Define a CSS custom-property palette at `:root` for brand colours, spacing scale, border-radius, and shadow values.
- Use a flat, predictable selector strategy — no deep nesting that would break if the HTML structure changes.

### Mandatory CSS Hooks (deterministic — must match exactly)

| Selector | Purpose |
|---|---|
| `.dashboard` | Top-level grid/flex container for all cards |
| `.project-card` | Individual card wrapper — must include `border-radius` and `box-shadow` |
| `.status-badge` | Inline badge for On Track / At Risk / Blocked |
| `.priority` | Priority label (High / Medium / Low) |

### Status Badge Colour Coding

| Status | Suggested colour treatment |
|---|---|
| On Track | Green background, dark-green text |
| At Risk | Amber/yellow background, dark-amber text |
| Blocked | Red background, dark-red or white text |

All colour combinations must meet **WCAG 2.1 AA contrast ratio (≥ 4.5:1)** for text on the badge background.

### Priority Treatment

- High priority: bold label, optional red left-border accent on the card.
- Medium priority: normal weight.
- Low priority: muted/secondary colour.

The Designer may use a modifier class pattern (`.priority--high`, `.priority--medium`, `.priority--low`) or `data-priority` attribute selectors; this choice must be communicated to the Orchestrator so the Coder uses the matching pattern in HTML.

### Responsive Layout

| Breakpoint | Layout |
|---|---|
| `< 600 px` | Single column, full-width cards |
| `600 px – 1023 px` | Two-column grid |
| `≥ 1024 px` | Three-column grid |

Use CSS Grid (`grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))`) for fluid responsiveness without media-query fragility.

### Typography & Spacing

- Use a system font stack (`-apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`).
- Card internal padding: at least `1.25rem`.
- Card gap: at least `1rem`.
- Page max-width: `1200px`, centred.

### Accessibility

- Ensure focus styles are visible (`:focus-visible` outline, not `outline: none`).
- The `.status-badge` must not rely on colour alone — include a short text label.
- Sufficient touch target size for any interactive elements (≥ 44 × 44 px).

### Visual Polish

- Subtle card hover state: slightly elevated `box-shadow` and `transform: translateY(-2px)`, with `transition: transform 0.15s ease, box-shadow 0.15s ease`.
- Page background: a light neutral (e.g., `#f5f7fa`) to contrast with white cards.
- Dashboard header: title "Project Pulse" in a larger weight, subtitle/tagline optional.

---

## 8. Coder Responsibilities

The Coder agent is assigned **`app/index.html`**, **`app/project-data.json`**, and **`.vscode/launch.json`**. The `app/styles.css` file must not be modified.

### `app/project-data.json` — Data Schema

Top-level structure:

```json
{
  "projects": [ "…6 entries…" ]
}
```

Each entry must contain:

| Field | Type | Allowed values / notes |
|---|---|---|
| `name` | string | Human-readable project name |
| `owner` | string | GitHub handle or full name |
| `status` | string | `"On Track"`, `"At Risk"`, or `"Blocked"` |
| `recentActivity` | string | Short prose sentence describing latest activity |
| `priority` | string | `"High"`, `"Medium"`, or `"Low"` |
| `description` | string | One or two sentence summary |
| `lastUpdated` | string | ISO 8601 date, e.g. `"2025-07-10"` |

Distribute statuses and priorities across the 6 entries for realistic variety (e.g., 3 On Track, 2 At Risk, 1 Blocked; 2 High, 3 Medium, 1 Low).

### `app/index.html` — HTML Structure

- Valid HTML5 (`<!DOCTYPE html>`, `lang="en"`, `charset="UTF-8"`, `viewport` meta tag).
- `<link rel="stylesheet" href="styles.css">` in `<head>` (relative path, no `/` prefix).
- Document title must include "Project Pulse".
- A `<main class="dashboard">` element as the card grid container.
- A `<script>` block that uses `fetch('project-data.json')` to load data and dynamically renders one `.project-card` per entry.
- Each card must include: project name (heading), owner, `.status-badge`, `.priority`, `recentActivity`, `description`, and `lastUpdated`.
- Apply modifier classes on `.priority` (e.g., `.priority--high`) as specified by the Designer.
- Graceful error handling: if `fetch` fails, display a visible error message inside `<main>`.

### `.vscode/launch.json` — Launch Configuration

- Strict JSON — no comments, no trailing commas.
- `"version": "0.2.0"`.
- One configuration:
  - `"name": "Run Project Pulse Dashboard"`
  - `"request": "launch"`
  - `"file": "${workspaceFolder}/app/index.html"`
  - `"cwd": "${workspaceFolder}/app"`
- Must open `index.html` directly so the learner sees the dashboard.

---

## 9. Edge Cases

| Edge case | Where to handle | Mitigation |
|---|---|---|
| `fetch` fails on raw `file://` URL | `app/index.html` JS | Catch the rejection; render a human-readable error in `<main>` |
| JSON has a missing field on one entry | `app/index.html` JS | Guard each field access with a fallback (e.g., `project.owner ?? "Unknown"`) |
| Empty `projects` array | `app/index.html` JS | Render an empty-state message: "No projects found." inside `.dashboard` |
| Status value outside the known set | `app/styles.css` / HTML | Fall back to a neutral grey `.status-badge` background |
| Priority value outside the known set | Same | Render the raw priority string without a modifier class |
| Long project name or description overflowing the card | `app/styles.css` | Use `overflow-wrap: break-word` and `line-clamp` on the description |
| Narrow mobile viewport (< 360 px) | `app/styles.css` | No element has a fixed width wider than `100vw` |
| `launch.json` type not supported in Codespace | `.vscode/launch.json` | Use the Codespace-compatible browser launch type (see Open Questions) |
| Colour-only status distinction | `app/styles.css` | Status badges include text labels, not icon-only |
| Screen-reader card order | `app/index.html` | Use semantic heading levels (e.g., `<h2>` for project name per card) |

---

## 10. Validation Expectations

### `app/project-data.json`
- [ ] File exists at `app/project-data.json`.
- [ ] `JSON.parse` succeeds (no syntax errors).
- [ ] Top-level key is `projects` and its value is an array.
- [ ] Array has ≥ 4 entries (plan targets 6).
- [ ] Every entry contains `name`, `owner`, `status`, `recentActivity`, `priority`, and `description`.
- [ ] `status` values are one of: `On Track`, `At Risk`, `Blocked`.
- [ ] `priority` values are one of: `High`, `Medium`, `Low`.

### `app/index.html`
- [ ] File exists at `app/index.html`.
- [ ] Contains the string `Project Pulse` (title and/or visible heading).
- [ ] Contains `<link rel="stylesheet" href="styles.css">`.
- [ ] Contains `fetch('project-data.json')` or equivalent.
- [ ] Contains `class="dashboard"` on the main container.
- [ ] Contains `class="project-card"` on card elements (static or dynamic).
- [ ] Contains `status-badge` and `priority` class references.
- [ ] Renders cards with owner, status, recent activity, and priority when opened via a local server.

### `app/styles.css`
- [ ] File exists at `app/styles.css`.
- [ ] Contains `.dashboard` rule with grid or flex layout.
- [ ] Contains `.project-card` rule with `border-radius` and `box-shadow`.
- [ ] Contains `.status-badge` rule with distinct colour treatments per status.
- [ ] Contains `.priority` rule (and modifier variants if used).
- [ ] Cards render visually distinct from the page background.
- [ ] Layout reflows to single column on a narrow viewport.

### `.vscode/launch.json`
- [ ] File exists at `.vscode/launch.json`.
- [ ] Parses as valid strict JSON.
- [ ] Contains a configuration with `"name": "Run Project Pulse Dashboard"`.
- [ ] Contains a reference to `index.html`.
- [ ] `cwd` is set to `${workspaceFolder}/app`.
- [ ] Running the configuration opens the dashboard page, not a directory listing.

### End-to-end
- [ ] Open the Run Project Pulse Dashboard launch configuration.
- [ ] Dashboard page renders with ≥ 4 visible project cards.
- [ ] Each card shows name, owner, status badge, priority, and recent activity.
- [ ] Status badges display distinct colours for at least two different statuses.
- [ ] Page is readable on a simulated mobile viewport (360 px wide).

---

## 11. Open Questions

| # | Question | Impact | Suggested resolution |
|---|---|---|---|
| 1 | **Launch config type in Codespace:** The correct type may be `"type": "chrome"` or `"type": "msedge"` with a `url: "http://localhost:5500/index.html"` if Live Server is installed. | `.vscode/launch.json` may silently fail if the wrong type is used. | Coder agent should check what VS Code extensions are available in the dev container. A fallback is to document that the user should install the Live Server extension (ritwickdey.LiveServer). |
| 2 | **`fetch` vs. inline data:** A raw `file://` URL will block `fetch` due to CORS. Should the HTML embed data inline as a JS variable as a fallback? | Affects learner experience if they open the file directly. | The plan opts for `fetch` (matches the brief), with a user-visible error message as the fallback. |
| 3 | **Priority modifier class pattern:** The Designer may choose `.priority--high` BEM-style modifiers or `[data-priority="high"]` attribute selectors. The Coder must know which before writing the card rendering loop. | A mismatch means priority styling silently does not apply. | Phase 0 contract must explicitly specify the pattern before Phase 1 starts. |
| 4 | **`lastUpdated` field:** The brief lists `name`, `owner`, `status`, `recentActivity`, `priority` as required JSON fields. `lastUpdated` is included for richness but not explicitly required by grading. | Minor — adding it is harmless. | 6-entry plan includes `lastUpdated`; can be removed without touching other files if needed. |
| 5 | **Number of sample projects:** The brief says "at least 4–6." The plan targets exactly 6. | Minor. | 6 is the safe upper bound for grading compliance and visual richness. |
