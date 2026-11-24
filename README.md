# EVENT HORIZON V5

### Guarded HTML Optimizer · Browser-Based Development Workbench

**Event Horizon V5** is a self-contained, offline-first HTML development tool for **minifying**, **beautifying**, **auditing**, **previewing**, and **exporting** web documents. It combines a two-pane code editor with an optimization pipeline, per-pass telemetry, a heuristic accessibility/SEO auditor, and a sandboxed rendering preview.

V5 evolves the original **Event Horizon Phase 4** application. Its primary engineering objective is to **reduce the risk of breaking a working page during optimization while preserving the existing editing experience and features**. It favors conservative transformations and an optional DOM-equivalence check over aggressive, unverified compression.

> **Important:** Event Horizon is a browser-native, token-aware HTML transformer—not a complete HTML/CSS/JavaScript abstract syntax tree (AST) compiler. Its DOM guard detects many structural changes, but it cannot prove full behavioral or visual equivalence.

## At a glance

| Property | V5 |
| --- | --- |
| Application type | Standalone HTML/CSS/JavaScript workbench |
| Entry point | `Event_Horizon_V5.html` |
| Installation | Open the file in a modern browser |
| Runtime dependencies | None: no npm packages, CDNs, backend, or API keys |
| Connectivity | Core editor and compiler work offline; previewed documents may request external resources |
| Input | HTML typed/pasted into the editor or imported from `.html`, `.htm`, `.txt` |
| Output | Optimized/beautified HTML, Markdown, React-oriented JSX, JSON telemetry |
| Processing | Browser Web Worker when available; asynchronous main-thread fallback |
| Storage | Browser `localStorage`, when permitted |
| Preview isolation | `<iframe sandbox="allow-scripts">`, without `allow-same-origin` |
| Status | V5; browser integration and in-app checks performed (see [Testing](#testing)) |

## Quick start

1. Download **`Event_Horizon_V5.html`**.
2. Open it in **Chrome, Edge, Firefox, or another reasonably modern browser**. Chrome/Edge on Windows are a practical starting point.
3. Paste your HTML into **SOURCE.html**. Alternatively, select **IMPORT HTML** or drag a file onto the source pane.
4. Select **MINIFY** or **BEAUTIFY**; leave **DOM Equivalence Guard** enabled for a safer default.
5. Select **COMPILE DATA**. To compile following edits automatically, enable **Live Auto-Compile**.
6. Inspect the **COMPILED OUTPUT**, **Metrics**, **Telemetry Engine**, **A11Y & SEO Auditor**, and **Render** tabs.
7. Select **SAVE** to download the current output or **COPY** to put it on the clipboard. Use **MD** or **JSX** for other export views; **HTML** returns to the compiled HTML without needing to recompile.

**No build step is required.** If browser restrictions affect `file://` storage or Workers, you can run a local server in the folder containing the HTML:

```bash
python -m http.server 8000
```

Then visit `http://localhost:8000/Event_Horizon_V5.html`.

### An example input

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Demo</title>
  <style>
    .used { color: rebeccapurple; }
    .not-used { color: red; }
  </style>
</head>
<body>
  <main class="used">
    <h1>Hello, Event Horizon</h1>
    <p>Keep <strong>important</strong> spaces intact.</p>
  </main>
</body>
</html>
```

In **MINIFY** mode, Event Horizon attempts safe attribute/whitespace reductions. When the document appears static and the selector is simple, it can remove unused rules such as `.not-used`. In the example, the whitespace around `<strong>` is meaningful and should remain intact. Actual output depends on selected options and whether the DOM guard accepts the candidate.

## Interface and features

### 1. Source and output editors

- Separate **editable source** and **read-only compiled output** panes.
- HTML-oriented syntax highlighting and synchronized scrolling between the code and highlight layers.
- Line-number gutters with **windowed rendering** to reduce the cost of very large files.
- Four-space indentation using `Tab` in the source editor.
- Optional automatic insertion of closing HTML tags.
- Mouse/pointer-draggable divider on desktop and keyboard-adjustable divider.
- Stale-output indicator when the source/options change before recompilation.
- Mobile and narrow-screen stacking, accessible focus styles, and reduced-motion support.
- Safe degradation to plain-text highlighting for input over **120,000 characters**.

### 2. Compilation modes

| Mode | Purpose | Preservation strategy |
| --- | --- | --- |
| **MINIFY** | Reduce redundant bytes while keeping markup functional | Selective, guarded operations; skips higher-risk rewrites |
| **BEAUTIFY** | Improve source readability and indentation | Preserves inline text and raw/preformatted content where possible |

These modes use the same workbench and analysis panels. Changing mode can trigger compilation when the source is populated.

### 3. Minification controls

| Control | Default | What it does |
| --- | --- | --- |
| **HTML5 Optional Closing Tags** | Off | Attempts omission of specific optional end tags, subject to the DOM guard |
| **Remove Safe Attribute Quotes** | On | Unquotes compatible attribute values; leaves values needing quotes intact |
| **Collapse HTML Booleans** | On | Shortens selected Boolean attributes such as `disabled="disabled"` |
| **Safe CSS & JS Whitespace** | On | Applies conservative formatting reductions; avoids broad regex rewriting of JavaScript |
| **Conservative Unused CSS Pruning** | On | Prunes only certain simple unused selectors in apparently static documents |
| **Remove Redundant Type Attrs** | On | Removes recognized default script/style type attributes |
| **Clean Empty Class/Style Attrs** | On | Removes empty `class` and `style` attributes in supported forms |

**CSS pruning is intentionally limited.** Scripts, template syntax, and some forms of dynamic markup disable the risky pruning pass. Complex selectors are generally retained. This can yield slightly larger files than an aggressive minifier, in exchange for fewer unsafe guesses.

**JavaScript handling is intentionally conservative.** V5 does not claim to parse and reprint full ECMAScript. It preserves URLs, regex literals, and template-literal content in the tested cases, rather than applying the old punctuation-stripping transformation indiscriminately.

### 4. Live pipeline and safety controls

| Control | Default | Purpose |
| --- | --- | --- |
| **Live Auto-Compile** | Off | Debounced compilation after edits, approximately 450 ms after input settles |
| **DOM Equivalence Guard** | On | Compares parsed source/output structure, attributes, and significant text; restores source if a mismatch is found |
| **Auto-Close HTML Tags** | On | Adds a matching end tag for some manually entered opening tags |

The **DOM guard** is an additional defense—not a formal semantic equivalence proof. The optimizer may also leave some safe compression opportunities unused. If the guard detects a mismatch, the compilation telemetry reports a **DOM safety rollback** and the editor displays the original source as the protected result.

### 5. Metrics and telemetry

The dashboard provides:

- **Data Saved:** difference in UTF-8 bytes between the current input and accepted HTML output. Growth is reported as added bytes rather than hidden.
- **Gzip Size:** size measured by the browser's `CompressionStream('gzip')` implementation when available. Otherwise the value is **N/A**, not a fabricated estimate.
- **Compile Time:** duration of the transformation/measurement pipeline, in milliseconds. This is **not** a browser runtime or page-load benchmark.
- **DOM Statistics:** element count and maximum parent-chain depth obtained through a parsed document.
- **Audit Grade:** a simplified A/B/C/F quality indicator based on findings from built-in heuristic checks.

The **Telemetry Engine** lists optimization passes and net byte changes. Warnings report skipped pruning, protected template literals, and safety rollbacks. Use **EXPORT TELEMETRY JSON** for a machine-readable report containing mode, sizes, elapsed time, passes, warnings, and audit messages.

### 6. Accessibility, SEO, and security heuristics

The built-in auditor uses `DOMParser` and checks for selected issues, including:

- Missing page language, title, meta description, or top-level heading.
- Missing `alt` attributes on images.
- Links without meaningful text/accessibility labels or without `href`.
- Buttons and form controls without detectable labels.
- Skipped heading levels.
- `target="_blank"` links without `rel="noopener"`.

**The audit is intentionally limited.** Its grade is **not** a Lighthouse score, a WCAG conformance certificate, a complete HTML validator, or a security scan. It does not inspect every accessibility tree, CSS contrast ratio, JavaScript-driven state, or runtime vulnerability.

### 7. Live render preview

The **Render** tab displays compiled HTML using a restricted iframe:

```html
<iframe sandbox="allow-scripts" referrerpolicy="no-referrer"></iframe>
```

Scripts in the supplied page may run within that sandbox. The iframe is **not** granted `allow-same-origin`, helping separate it from the workbench's own document and storage.

**Security warning:** Sandboxing does **not** prevent every network request or all possible effects of untrusted content. HTML that imports images, styles, fonts, scripts, or other resources may contact external servers. Avoid previewing sensitive or hostile documents without evaluating the risks.

### 8. Imports, exports, and state management

| Action | Result |
| --- | --- |
| **IMPORT HTML** | Reads a local `.html`, `.htm`, or `.txt` file into the source editor |
| **Drag and drop** | Imports a file dropped onto the source pane |
| **SAVE** | Downloads the currently visible HTML, Markdown, or JSX output |
| **COPY** | Copies the currently visible output, with a selection-based fallback |
| **MD** | Views a Markdown conversion of compiled HTML |
| **JSX** | Views a React-oriented representation of compiled HTML |
| **HTML** | Returns to the preserved compiled HTML view |
| **OUTPUT → SOURCE** | After confirmation, replaces the editable source with the compiled HTML and recompiles |
| **EXAMPLE** | After confirmation, replaces the source with the bundled demonstration page |
| **EXPORT TELEMETRY JSON** | Downloads a structured report of the last accepted compilation |
| **RUN BUILT-IN TESTS** | Executes nine compiler-focused checks and displays PASS/FAIL results |

The file importer imposes an **8 MiB maximum**. Copying output, saving it to disk, or explicitly using **OUTPUT → SOURCE** is recommended before destructive edits.

**Export details:**

- The HTML download is named `optimized.min.html` in minify mode or `beautified.html` in beautify mode.
- Markdown downloads as `optimized.md`; JSX downloads as `optimized.jsx`.
- Markdown export preserves useful text structure but cannot express arbitrary CSS, JavaScript, or complex HTML behavior.
- JSX export uses element-level React-oriented output for relatively simple documents. For complex pages (for example, scripts or SVG), it generates a component with an **isolated `srcDoc` iframe**, rather than silently dropping unsupported document behavior.
- Switching export views does **not** overwrite the compiled HTML held by the application.

### Keyboard accessibility

| Shortcut | Action |
| --- | --- |
| `Ctrl+Enter` / `Cmd+Enter` | Compile source |
| `Ctrl+S` / `Cmd+S` | Download the currently selected output |
| `Tab` inside source editor | Insert four spaces |
| `ArrowLeft` / `ArrowRight` when editor divider is focused | Resize the source/output split |
| `Enter` / `Space` on a focused bottom tab | Open that tab |

## Under the hood

V5 is a **single-file application** with three major subsystems:

```text
Event_Horizon_V5.html
│
├── Presentation layer
│   ├── Dark / animated Event Horizon UI
│   ├── Side-by-side source and output editors
│   ├── Editor highlighting and virtualized line-number gutters
│   ├── Mode/options and file actions
│   └── Metrics, telemetry, auditor, and preview tabs
│
├── Compilation engine (compileEngine)
│   ├── Quote-aware HTML tag scanner and tokenizer
│   ├── Special/raw-content preservation
│   ├── Selected attribute transformations
│   ├── Structural whitespace reduction
│   ├── Optional end-tag handling
│   ├── Conservative CSS and JavaScript passes
│   ├── Gzip measurement, if supported
│   └── Per-pass metrics and warnings
│
└── Application controller
    ├── Web Worker orchestration and fallback
    ├── Compile generation IDs / stale-result rejection
    ├── DOM comparison and rollback
    ├── DOMParser-driven heuristic auditor
    ├── HTML / Markdown / JSX view management
    ├── Sandboxed preview
    └── Local storage, import/export, and keyboard interactions
```

### Compilation lifecycle

```text
Input / file import
       │
       ▼
Collect mode and checkbox options
       │
       ▼
Queue job (Web Worker if available)
       │
       ▼
Token-aware transformations + telemetry
       │
       ▼
Gzip measurement where supported
       │
       ▼
Reject outdated compile generation?
       ├── Yes ──► Ignore stale result
       └── No
             │
             ▼
       DOM guard enabled?
             ├── No ───► Accept candidate
             └── Yes
                   │
                   ▼
             Parsed-DOM comparison
                   ├── Mismatch ──► Restore source, warn
                   └── Match ─────► Accept candidate
                                        │
                                        ▼
                          Update output, metrics, audit,
                          telemetry and sandboxed preview
```

### Why the Web Worker matters

Compilation can run on a browser **Web Worker**, keeping the primary UI more responsive. Work is tagged with a monotonically increasing job ID, and results from older jobs are discarded. If Workers cannot be started, the same compiler is called through a Promise on the main thread.

The fallback improves compatibility but **does not provide equivalent parallelism**: large jobs may temporarily block UI responsiveness, and the 8 MiB import cap is not a guarantee that all documents compile quickly.

### Local persistence and privacy

The app stores its editable source and options **locally in the browser**, when `localStorage` is allowed:

| Key | Contents |
| --- | --- |
| `quantum_event_horizon_state` | Source HTML (the same key as the original Phase 4 app) |
| `quantum_event_horizon_settings_v5` | Selected mode and checkbox states |

Source autosave is debounced by approximately **350 ms**. Nothing in the compiler requires an account, API token, or server upload. Private/incognito modes, blocked storage, quota limits, browser-origin changes, and browser data clearing can interrupt persistence.

> **Use downloadable HTML as your durable backup.** Browser storage is a convenience, not version control or guaranteed long-term storage.

## V5 vs. Phase 4

| Area | Phase 4 | V5 |
| --- | --- | --- |
| UI and look | Event Horizon dark holographic interface | Retained, with responsive and accessibility refinements |
| Editor | Syntax highlighting, line numbers, auto-closing, resizer | Retained; stronger scrolling and large-file handling |
| HTML transformation | Broad regular-expression transformations | More token-aware, conservative transformations |
| JavaScript minification | Potentially destructive regex-based rewrites | Conservative whitespace handling; risky constructs generally retained |
| CSS pruning | Aggressive selector removal | Narrow, static-only selector pruning and skip warnings |
| Beautification | Basic HTML reformatting | Preserves more whitespace-sensitive/raw content |
| Safety validation | No equivalent integrated DOM rollback | Optional structural DOM comparison and rollback |
| Compiler responsiveness | Worker pipeline | Worker pipeline plus stale-job protection and main-thread fallback |
| Persistence | Source in local storage | Legacy source key retained; settings persisted separately |
| Export views | Markdown/JSX could overwrite visible compiled result | Non-destructive HTML/MD/JSX switching |
| File handling | Copy/save | Copy/save plus import, drag/drop, output-to-source, and JSON telemetry |
| Testing | No integrated check button in supplied Phase 4 | Nine runnable built-in checks, plus external Playwright coverage |

These are changes in architecture and tested workflows, **not** a claim that V5 is faster or produces smaller output for every possible source file. Phase 4 may occasionally achieve higher compression by performing transformations that V5 intentionally avoids.

## Testing

### In-app self-tests

Select **RUN BUILT-IN TESTS** from the sidebar. The nine checks cover:

1. Preservation of inline whitespace.
2. Quoted greater-than signs in attributes.
3. Simple unused-CSS pruning in static markup.
4. Preservation of dynamically referenced CSS.
5. JavaScript strings and regular expressions.
6. Template-literal preservation.
7. Preformatted text in beautify mode.
8. Nonempty React JSX export.
9. Markdown heading conversion.

Results appear in **Telemetry Engine** as individual PASS/FAIL rows.

### Browser integration tests

The companion `test_event_horizon_v5.py` uses **Playwright + Chromium**. The test session for this V5 artifact passed **16 of 16 integration checks**, including the built-in **9 of 9** checks. Coverage included worker/fallback compilation, HTML/CSS/JS preservation cases, output views, import/download actions, storage behavior, DOM rollback, a narrow mobile viewport, and uncaught JavaScript exceptions.

To prepare a Python environment:

```bash
python -m pip install playwright
python -m playwright install chromium
```

The existing companion test script was originally written to load the HTML from **`/mnt/data`** in the generation environment. Before running it from a different repository directory, update its `ROOT` assignment and the absolute HTML/screenshot/results paths to the desired local directory; then execute:

```bash
python test_event_horizon_v5.py
```

This test record is **evidence of selected scenarios working in one browser environment**. It does not certify universal HTML equivalence, complete accessibility, or performance across devices. For important production pages, add visual screenshot diffs, actual interaction tests, and target-browser checks.

## Compatibility and limitations

| Situation | Expected behavior or limitation |
| --- | --- |
| Modern Chromium-based browsers | Recommended primary environment |
| Other modern browsers | Likely functional, but not exhaustively tested for this release |
| `CompressionStream` unavailable | Gzip dashboard value shows **N/A** |
| Web Worker blocked or unavailable | Main-thread fallback processes jobs |
| `localStorage` blocked | Compiler still works; source/options may not persist |
| Large documents | Editor highlighting reduces work; huge compilations can still be expensive |
| Files above 8 MiB | Import action rejects them |
| JavaScript-heavy sites | Conservative optimization keeps more code and may skip CSS pruning |
| CSS with dynamic selectors, custom properties, complex `url()` syntax | May remain unminified or unpruned |
| JSX conversion of full web pages | Not a guaranteed direct React migration; complex inputs use iframe output |
| Browser rendering and semantics | DOM guard is necessary-but-insufficient for complete equivalence |
| Third-party resources in preview | May still generate requests and execute scripts in the isolated frame |

### Troubleshooting

**The output is identical to the input.** The input may already be compact, optimization may be deliberately skipped, or the DOM guard may have rejected a candidate. Open **Telemetry Engine** and look for a rollback/skipped-pass warning.

**My CSS rule disappeared.** Check whether static-only pruning was enabled and whether the selector was not present in source markup. Restore the source, disable **Conservative Unused CSS Pruning**, and recompile. For dynamic applications, do not assume that a class absent from initial HTML is unused.

**The preview doesn't look like my deployed website.** The preview is an isolated document with different origin and relative-resource resolution characteristics. External files, routing, storage, embedded permissions, and runtime dependencies may not behave as they do on your production origin.

**Saved edits disappeared.** Check browser storage restrictions or whether the file moved to a different URL/origin. Download your HTML regularly; use Git or another source-control solution for important projects.

**The editor becomes slow on a huge page.** Turn off live compilation, keep Worker support enabled, and divide very large projects into smaller source files where possible. Highlighting has a size cutoff, but compilation still processes the complete string.

**I need maximal minification.** V5 prioritizes safe best-effort transformations. For production pipelines needing advanced ECMAScript/CSS compression, use dedicated parser-backed tools and regression tests in addition to this workbench.

## Design principles and possible next steps

The V5 design deliberately emphasizes:

- **No feature removal:** retain the Phase 4 editors, modes, metrics, audits, preview, and core controls.
- **Conservative defaults:** prefer a missed byte-saving opportunity over a broken script or DOM.
- **Visible behavior:** report warnings, byte deltas, and safety rollbacks rather than implying every pass succeeded.
- **Offline portability:** keep a usable single-file application with no build service.
- **Progressive enhancement:** Worker acceleration, gzip support, and browser storage may degrade gracefully.

Future development candidates—not implemented in V5—include parser-backed HTML/CSS/JavaScript optimization, screenshot-based visual-diff verification, configurable optimization profiles, worker cancellation, a richer code editor with undo history, cross-browser automated CI, accessible contrast analysis, and source-map-aware bundling.

## Project files

```text
.
├── Event_Horizon_V5.html          # Standalone application
├── README.md                      # This documentation
└── test_event_horizon_v5.py       # Optional Playwright verification harness
```

The HTML file is the **only mandatory runtime artifact**. Documentation and test dependencies are optional. Keep a backup of Phase 4 if you want to compare behavior or compression output directly.

## Licensing

A software license was **not specified in the supplied source**. Before publishing this repository for reuse, choose a license appropriate to the original code's ownership and contribution history; do not assume that public availability automatically grants permission to redistribute or modify it.

---

**Event Horizon V5** — a practical browser-based HTML optimizer focused on **fidelity, observability, offline usability, and gradual, regression-conscious improvement**.
