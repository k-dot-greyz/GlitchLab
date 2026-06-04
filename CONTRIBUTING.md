# Contributing to GlitchLab 🎨

Welcome to **GlitchLab** — an experimental browser lab for real-time glitch art: stackable WebGL effects, MIDI-mappable parameters, and export pipelines. This repo is part of the **GlitchWorks** constellation; product overview lives in [README.md](./README.md). Task breakdown: [tasks.md](./tasks.md).

---

## Repository layout

| Path | Purpose |
|------|---------|
| [README.md](./README.md) | Product spec, stack, and architecture overview |
| [CONTRIBUTING.md](./CONTRIBUTING.md) | Contribution standards (this file) |
| [tasks.md](./tasks.md) | Epics and implementation checklist |
| [dex-entry.md](./dex-entry.md) | Dex metadata stub (status, tags, deploy target) |
| `src/` *(planned)* | TypeScript app entry, WebGL canvas, effect stack host |
| `src/effects/` *(planned)* | Isolated glitch effect modules (shader + params) |
| `src/core/` *(planned)* | Pipeline orchestration, state export/import, validation |
| `src/ui/` *(planned)* | Controls, preset UI, MIDI mapping surface |
| `schemas/` *(planned)* | JSON Schema for lab presets, effect stacks, and MIDI maps |
| `tests/` *(planned)* | Unit tests per effect; integration tests for pipeline |
| `tests/fixtures/` *(planned)* | Golden images and minimal valid/invalid config samples |
| `public/` *(planned)* | Static assets (noise textures, default displacement maps) |
| `docs/` *(planned)* | Product-facing docs only (shader API, preset format, MIDI map) |

**Planned scaffold** (create as code lands; do not commit monorepo runbooks here):

```
GlitchLab/
├── README.md
├── CONTRIBUTING.md
├── tasks.md
├── dex-entry.md
├── schemas/           # lab-config, effect-stack, midi-map
├── src/
│   ├── core/          # pipeline, exportState / loadState
│   ├── effects/       # one folder per effect plugin
│   └── ui/
├── tests/
│   └── fixtures/
├── public/
└── docs/              # product docs only
```

**Local dev** *(once scaffolded)*: `npm install` → `npm run dev`. Ports and deploy URLs belong in env or Vite config — never hardcoded in effect logic.

---

## 1. Submodule boundary: pure code and product docs

GlitchLab may be checked out inside parent workspaces (e.g. `dev-master` as `dex/09-repos/GlitchLab`). This upstream repository must stay clean.

### Boundary violation rule

**Do not commit** internal parent-workspace guides, zenOS agent notes, monorepo SOPs, submodule bump runbooks, or fork-specific setup into this repo.

| Allowed here | Belongs in superproject (`dev-master`) |
|--------------|----------------------------------------|
| Source, shaders, tests, schemas | `dex/03-docs/guides/` (submodule workflows, agent RAM) |
| `README.md`, `CONTRIBUTING.md`, `tasks.md` | Switchboard / dex routing docs |
| `docs/` describing GlitchLab behavior | Private fork runbooks |

**Allowed artifacts:** JSON/YAML under `schemas/` and preset files when they define **lab behavior** (effect stacks, MIDI maps), not monorepo orchestration.

---

## 2. GlitchWorks Agnostic Architecture Protocol (lab edition)

Experimental code fails loudly in dev and safely in prod. Design every effect as a **plugin** with explicit contracts, serializable state, and validated inputs.

### 2.1. Experimental safety

* **Isolate blast radius:** New effects and shaders ship behind a dedicated module boundary (`src/effects/<name>/`). Do not mutate global WebGL state outside the effect’s pass.
* **Opt-in danger:** Destructive paths (databend simulation, large buffer rewrites) must be gated by explicit UI or config flags; default presets stay non-destructive to source assets.
* **Resource caps:** Document and enforce reasonable limits (texture size, stack depth, export duration) at the pipeline edge; reject oversize inputs before GPU upload.
* **No silent corruption:** Invalid configs or shader compile errors must surface typed errors in the UI — never a blank canvas without feedback.

### 2.2. Zero hardcoding (dynamic state configuration)

* No magic ports, CDN URLs, or filesystem paths in domain logic.
* Vite dev server, preview, and `glitchlab.live` deploy targets live in config/env.
* Effect defaults live in schema-backed presets, not scattered literals in GLSL strings.

### 2.3. Modular plugin architecture (effect stack)

* **One effect, one module:** Each glitch algorithm exports a small contract: `id`, `params` schema, `apply(gl, state)`, and optional `dispose()`.
* **Ordered stack:** The pipeline composes effects in user order; reordering must not require rewriting unrelated effects.
* **Polymorphism:** UI and MIDI layers depend on abstract parameter descriptors, not concrete effect classes.

### 2.4. Open piping (strict boundaries)

* Cross-layer updates (UI → stack → export) use explicit message shapes (typed events or JSON payloads), not shared mutable globals.
* MIDI CC mapping flows through a dedicated map object validated against schema — not ad hoc property writes on the canvas.

### 2.5. Boundary validation (hostile edge)

* Validate imported images, preset JSON, and MIDI map files **before** merging into runtime state.
* Shader uniforms: clamp and type-check at the JS boundary; GLSL should assume hostile numeric input.

### 2.6. State hydration and dehydration (dynamic state export)

* **Rule:** The lab must export and resume truth as JSON.
* **Application:** Effect stack order, per-effect parameters, and global settings (input source, export format) serialize via `exportState()` / `loadState(payload)` on the core pipeline — suitable for localStorage, file download, or future cloud preset sync.
* Presets on disk must validate against `schemas/` before `loadState` runs.

### 2.7. Graceful degradation

* Missing `ffmpeg.wasm`, WebMIDI, or camera permission → clear fallback (disable feature, keep preview alive).
* WebGL2 unavailable → structured error with recovery hints, not an uncaught exception.

### 2.8. Agnostic telemetry

* Emit structured events (effect applied, export started, validation failed) through an injectable logger; core modules must not assume Sentry, CI, or browser console only.

---

## 3. Quality gates (experimental code)

Run these before every PR. CI should mirror the same commands once workflows exist.

| Gate | Requirement |
|------|-------------|
| **Isolated effect tests** | Each effect under `src/effects/` has unit tests that mock WebGL where needed; no test requires the full UI shell unless integration-tagged. |
| **Pipeline integration** | At least one test covers stack order, `exportState` → `loadState` round-trip, and empty-stack behavior. |
| **Schema validation** | All committed preset/stack/MIDI JSON under `schemas/` or fixtures validates with the project validator (e.g. `ajv` + JSON Schema). CI fails on drift. |
| **Shader compile check** | Build or test step compiles GLSL for WebGL2; compile failures fail the gate. |
| **Lint & types** | `npm run lint` and `tsc --noEmit` (when TypeScript scaffold exists). |
| **Build** | `npm run build` produces a deployable artifact without warnings treated as errors in CI. |
| **Boundary audit** | `git diff --name-only` contains no monorepo-only markdown; no secrets or `.env` tracked. |

**Suggested commands** *(after scaffold)*:

```bash
npm install
npm run lint
npm run test          # unit + effect isolation
npm run test:schemas  # validate fixtures and example presets
npm run build
```

Add `test:schemas` to `package.json` when `schemas/` lands; until then, document new JSON shapes in `docs/` and validate manually in PR description.

---

## 4. Contribution workflow

### Step 1: Remotes

```bash
git remote -v
# If you use a fork, set origin to your fork and add upstream:
# git remote add upstream https://github.com/k-dot-greyz/GlitchLab.git
```

### Step 2: Branch

```bash
git fetch origin
git checkout -b feat/your-change origin/main
# Prefixes: feat/, fix/, docs/, refactor/, test/, chore/
```

### Step 3: Implement

* Match stack choices in [README.md](./README.md) (TypeScript, Vite, WebGL2, optional Astro UI).
* Add or update JSON Schema when introducing preset or stack fields.
* Keep experimental flags explicit in config, not commented-out dead code.

### Step 4: Pre-commit checklist

1. **Misplaced docs?** Move monorepo/agent guides out of this repo.
2. **Diff scope?** Only GlitchLab-relevant files; no parent `dex/` paths.
3. **Schemas?** New fields reflected in `schemas/` and fixtures.
4. **Tests?** Effect-isolated coverage for new algorithms.
5. **Commits:** [Conventional Commits](https://www.conventionalcommits.org/) — `type(scope): subject` (e.g. `feat(pixel-sort): add luminance horizontal pass`).

### Step 5: Push and open a PR

```bash
git push -u origin HEAD
gh pr create --repo k-dot-greyz/GlitchLab --base main \
  --title "feat(effects): short summary" \
  --body "$(cat <<'EOF'
## Summary
- …

## Test plan
- [ ] `npm run lint`
- [ ] `npm run test`
- [ ] `npm run test:schemas` (if applicable)
- [ ] `npm run build`
- [ ] Manual: load preset, tweak stack, export PNG/GIF
EOF
)"
```

---

## 5. Coding conventions

* **TypeScript:** Strict types for effect params and pipeline state; no `any` on public plugin contracts.
* **GLSL:** One file pair per effect where possible; document uniform names in the effect module README snippet under `docs/` when non-obvious.
* **UI:** Prefer CSS custom properties for CRT/glitch theme tokens; keep WebGL uniforms out of CSS.
* **MIDI:** Map CC → parameter via validated map JSON; support unmapped CC as no-op.
* **Comments:** Explain non-obvious GPU or byte-manipulation constraints only.

---

## 6. Related projects

* [GlitchWorks](https://github.com/k-dot-greyz/GlitchWorks) — parent constellation
* [midi-gem](https://github.com/k-dot-greyz/midi-gem) — optional hardware MIDI routing
* [glitch-that-shit](https://github.com/k-dot-greyz/glitch-that-shit) — browser extension sibling

---

*Unacceptable conditions will be glitched into submission — safely, with schemas and exportable state.*
