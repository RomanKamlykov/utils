# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static collection of small, single-file browser utility tools (JWT decoder, text-to-table, DDL-to-TypeScript, JSON diff, etc.), served as plain HTML — no build step, no bundler, no package.json.

## Running locally

```sh
npx http-server ./docs
```

There is no build, lint, or test command — files are served as-is.

## Architecture

- `docs/index.html` is the landing page: a Bootstrap navbar/list linking to every tool page. **Any new tool must be added here** or it's unreachable from the site.
- `docs/*.html` — one self-contained tool per file. Each is independently deployable/shareable; there is no shared JS/CSS file between them, so common helpers (e.g. `chunk`, `upperFirst`) are duplicated per-file rather than imported.
- No JS framework build: Vue 3 (`vue.global.js`) and Bootstrap 5.3.8 are pulled from CDN (`unpkg`/`jsdelivr`) directly in `<head>`. A couple of tools pull an extra CDN lib for their specific job (`js-yaml` in `stream-templates.html`, `grapesjs` in `newsletter-builder.html`).
- `tmp/` is a gitignored scratch directory — not part of the deployed site.

### Standard tool page structure

Every tool page (except `newsletter-builder.html`, which embeds the GrapesJS editor instead) follows the same template:

1. `<head>`: Bootstrap CSS + Vue 3 global build from CDN, `<title>` matching the tool name.
2. Bootstrap navbar with a brand link back to `/utils` (the site root).
3. `.container > #app`: heading, optional source/docs link, one or more `<textarea>`/`<input>` bound via `v-model`, and a `<pre>{{ result }}</pre>` showing computed output.
4. A single inline `<script>` block: `const {computed, createApp, ref} = Vue;`, plain helper functions declared at module scope, then `createApp({ setup() {...} }).mount('#app')` using the Composition API — state as `ref`, derived output as `computed`. No Vue SFCs, no reactivity outside of `ref`/`computed`/`watchEffect`.
5. Bootstrap JS bundle `<script>` at the end of `<body>`.

Recurring conventions worth reusing rather than reinventing:
- Auto-sizing textareas: `rows = computed(() => Math.min(Math.max(text.value.split('\n').length, 5), 20))`.
- Normalizing pasted input: wrap a `ref` in a `computed({get, set})` that strips `\r` (`val.replace(/[\r\n]+/g, '\n')`).
- Persisting input across reloads: `ref(localStorage.getItem(key) || '')` + `watchEffect(() => localStorage.setItem(key, value))`, keyed by `` `${location.pathname}:fieldName` `` (see `compare-objects.html`).
- File download from generated content: build a `Blob`, create an `<a download>` with `URL.createObjectURL`, click and revoke it (see `stream-templates.html`'s `onFileDumped`).
- All conversion/parsing logic is pure functions outside `setup()`; `setup()` only wires refs/computed to the DOM.

## Adding a new tool

1. Copy the structure of an existing simple tool (e.g. `docs/text-to-hex.html` or `docs/chrome-headers.html`) rather than starting from scratch.
2. Keep it a single self-contained HTML file in `docs/`.
3. Add a link to it in `docs/index.html`'s list.
