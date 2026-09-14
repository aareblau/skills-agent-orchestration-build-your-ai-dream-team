# Project Pulse dashboard implementation plan

## Project Pulse summary

Project Pulse is a lightweight static dashboard that helps Mona's contributors
quickly understand active projects, their owners, current status, recent
activity, and priority or risk level. The finished experience is a polished,
accessible card-based frontend served from the `app/` directory. Opening the
VS Code launch configuration must show `index.html` and the Project Pulse UI,
not a directory listing.

The dashboard will use local JSON data rather than a backend. Its data contract
is a top-level `projects` array. Every project object must provide `name`,
`owner`, `status`, `recentActivity`, and `priority`.

## Roles and file ownership

The Orchestrator coordinates the work, resolves ordering and handoffs, and
checks that the assigned agents stay within their ownership. The Orchestrator
does not implement dashboard files.

The Planner authors this plan, identifies dependencies and parallel work, and
does not implement the dashboard.

| Owner | Exact file assignment | Responsibility |
| --- | --- | --- |
| Designer | `app/styles.css` | Own the visual and UX specification in CSS: readable information hierarchy, responsive layout, status and priority presentation, spacing, color contrast, focus treatment, and polished card styling. The CSS must include `.dashboard`, `.project-card`, `border-radius`, and `box-shadow`. |
| Coder | `app/project-data.json` | Create valid static project data using the required top-level `projects` array and all five required fields on each project. Supply multiple representative projects so cards can be verified. |
| Coder | `app/index.html` | Build the accessible static dashboard markup and client-side rendering that reads `project-data.json`, uses the exact visible title `Project Pulse`, and creates a `.project-card` for each project. Link `styles.css`; render name, owner, status, recent activity, and priority. |
| Coder | `.vscode/launch.json` | Create strict JSON with no comments. Define the `Run Project Pulse Dashboard` configuration, run `python3 -m http.server 5500` with `cwd` set to `${workspaceFolder}/app`, and use `serverReadyAction` to open `http://localhost:%s/index.html`. |

## Ordered implementation steps

1. **Confirm the contracts before implementation.** The Orchestrator shares
   this plan with Designer and Coder. Confirm the app is static, that JSON is
   loaded over the local HTTP server rather than from the filesystem, and that
   Coder owns the three implementation artifacts plus the launch configuration.
2. **Prepare the data contract.** Coder creates `app/project-data.json` with a
   top-level `projects` array and multiple project objects. Each object includes
   non-empty `name`, `owner`, `status`, `recentActivity`, and `priority` values.
   Use consistent, contributor-friendly text so the UI can display every field
   directly.
3. **Define the visual system.** Designer creates `app/styles.css` for a
   polished frontend dashboard. Include responsive rules for narrow screens,
   a `.dashboard` layout container, and `.project-card` styling with
   `border-radius` and `box-shadow`. Establish visually distinct but
   understandable status/priority treatment, readable text, and visible
   keyboard focus states without relying on color alone.
4. **Build the dashboard.** Once the data shape is available, Coder creates
   `app/index.html`. Add the exact `Project Pulse` title, semantic page
   structure, a dashboard region, a loading/error message area, and a linked
   `styles.css`. Fetch or otherwise load `project-data.json` from the served
   app, validate that `projects` is an array, and render one
   `.project-card` per project with labels and visible values for name, owner,
   status, recent activity, and priority.
5. **Add a reliable preview launch.** Coder creates `.vscode/launch.json` as
   strict JSON (no comments or trailing commas). Its `Run Project Pulse
   Dashboard` configuration starts `python3 -m http.server 5500` from
   `${workspaceFolder}/app`; `serverReadyAction` opens
   `http://localhost:%s/index.html` so learners see the dashboard frontend.
6. **Integrate and review.** Designer reviews the rendered HTML with the final
   CSS, while Coder corrects markup/data/launch integration issues within the
   owned files. The Orchestrator coordinates findings only and requests changes
   from the responsible owner.
7. **Validate the completed dashboard.** Run the data, launch, browser,
   accessibility, and exercise checks listed below. Address blocking failures
   before handoff.

## Dependencies and parallel work decisions

### Can run in parallel

- Designer can create `app/styles.css` in parallel with Coder creating
  `app/project-data.json`; these files have no implementation dependency.
- Coder can create `.vscode/launch.json` in parallel with the data and CSS
  work because its command, working directory, name, and URL are fixed.
- Designer can define responsive and accessible CSS while Coder starts the
  semantic HTML shell, provided the agreed class names remain `.dashboard` and
  `.project-card`.

### Must remain sequential

- The exact JSON data shape must be settled before Coder completes JavaScript
  rendering in `app/index.html`; rendering depends on the top-level `projects`
  array and the five required field names.
- The browser integration review follows completion of HTML, CSS, JSON, and
  launch configuration because it verifies their interaction over HTTP.
- Final validation follows integration fixes. `bash scripts/validate-exercise.sh`
  is the final repository-level check, not a substitute for visible browser and
  accessibility review.

## Integration and edge-case considerations

- `fetch("project-data.json")` requires the HTTP preview. Opening
  `index.html` directly with a `file://` URL may prevent JSON loading; use the
  prescribed launch configuration for testing.
- Keep the relative data and stylesheet paths compatible with serving from
  `app/`. Do not make the page depend on a framework, package installation, or
  backend service.
- If the JSON request fails, JSON is malformed, the `projects` key is missing,
  or it is not an array, show a clear in-page error rather than leaving an
  indefinite loading state.
- Handle an empty `projects` array with a concise empty-state message. Avoid
  assuming a fixed project count, card order beyond JSON order, or a limited
  length for names and activity text.
- Escape or insert data as text, not untrusted HTML, when rendering project
  values. This keeps JSON content from becoming executable markup.
- Preserve all required fields even when a value is unusually long. Cards
  should wrap text and remain usable at narrow viewport widths.
- Use semantic landmarks and headings; give status and priority text labels in
  addition to visual badges. Ensure sufficient contrast, keyboard-visible
  focus, and a sensible reading order.
- The preview must target `index.html` through `serverReadyAction`; opening
  only the server root is not acceptable because it can display a directory
  listing instead of the dashboard.

## Validation expectations

1. **JSON contract:** Run
   `python3 -m json.tool app/project-data.json` and confirm parsing succeeds.
   Inspect that `projects` is top-level, is an array, and every project has
   `name`, `owner`, `status`, `recentActivity`, and `priority`.
2. **Static-file integration:** Inspect `app/index.html` to confirm it links
   `styles.css`, references or fetches `project-data.json`, contains the exact
   `Project Pulse` title, and renders cards using `project-card`. Confirm CSS
   contains `.dashboard`, `.project-card`, `border-radius`, `box-shadow`, and
   responsive styling.
3. **Launch configuration:** Run
   `python3 -m json.tool .vscode/launch.json`. Verify strict JSON, the
   configuration name `Run Project Pulse Dashboard`, command
   `python3 -m http.server 5500`, cwd `${workspaceFolder}/app`, and a
   `serverReadyAction` URL of `http://localhost:%s/index.html`.
4. **Browser launch and visible rendering:** In VS Code Run and Debug, choose
   **Run Project Pulse Dashboard** and start it. Confirm the browser opens
   `index.html`, shows the Project Pulse dashboard rather than a directory
   listing, loads multiple cards, and visibly displays every required project
   field. Stop the preview server after the check.
5. **Accessibility and responsive review:** Check heading order, landmark use,
   readable labels, contrast, non-color status/priority cues, keyboard focus,
   and keyboard navigation. Resize to a narrow viewport and confirm cards
   remain readable without clipped or overlapping content.
6. **Repository exercise check:** Run
   `bash scripts/validate-exercise.sh` from the repository root. Treat any
   failure as a blocking issue to investigate; record successful validation in
   the implementation handoff.

## Non-blocking open questions

- Which project names and realistic activity wording best match Mona's team's
  current work? Representative static entries are acceptable until a source of
  record is available.
- Should `priority` be described in the UI as priority, risk, or both? The data
  key remains exactly `priority` regardless of its contributor-facing label.
- Are there project-specific status terms or a preferred color palette the team
  wants later? The initial implementation should use clear text labels and an
  accessible neutral palette so these refinements do not block delivery.
