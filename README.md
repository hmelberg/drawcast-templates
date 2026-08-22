# drawcast-templates

Community-contributed template packs for [drawcast](https://github.com/hmelberg/drawcast) — the
official pack index. drawcast's editor can browse this repo's packs from its
Template packs panel ("Browse official packs…") and load any of them with one
click; anyone can also load a pack from this repo (or any other https URL) by
pasting its raw YAML URL into "Add pack from URL…".

Nothing here is fetched automatically. drawcast only reads this repo's
`index.json` when a user clicks "Browse official packs…", and only fetches
one pack's YAML when they click "Add".

## Format

A pack file is a multi-document YAML file: one header document, followed by
one **TemplateDoc** per template. This is the exact format drawcast's bundled
packs use (see [`src/scenes/packs/*.yaml`](https://github.com/hmelberg/drawcast/tree/main/src/scenes/packs)
in the main repo — `physics.yaml` is a good short reference) — a remote pack
is registered through the same `parsePack`/`registerPack` code path as a
bundled one, so there is no separate, laxer format for community packs.

### Header (first document)

```yaml
pack: my_pack_id       # lowercase snake_case, must be unique across all packs the user has loaded
title: My Pack
description: One line describing what this pack draws.
```

### Each template (one document per `---`)

```yaml
template: my_template_id   # lowercase snake_case, must not collide with any built-in or other loaded template
title: My Template          # optional, human-readable
version: 1
kit: 1                      # the sceneKit version this template's layout body was written against
status: ready                # "ready" (has a layout body) or "stub" (catalog entry only)
description: >-
  What this scene draws and WHEN an LLM should pick it — this is the routing
  text a compiler prompt sees, so be specific about the requests it should match.
params:
  type: object               # a JSON Schema for the template's params — content only, never coordinates
  properties:
    ...
element_ids:
  some_id: what this element is, in the rendered figure
examples:
  - request: "A natural-language request this template should satisfy."
    params: { ... }
layout: |
  // A JS function body: (params, kit, engines) => { drawables, labels, anchors, order }.
  // `kit` is drawcast's sceneKit (kit.stroke, kit.area, kit.text, kit.label,
  // kit.ellipse, kit.group, COLORS, CANVAS, SKETCH_MS, ...) — see sceneKit's
  // doc comments in src/scenes/kit.ts in the main repo. The canvas is a fixed
  // 1000×750 logical unit, Cartesian, y-up plane.
  const drawables = [], labels = [], anchors = {}, order = [];
  // ... build drawables/labels here ...
  return { drawables, labels, anchors, order };
```

A template registers **all-or-nothing per pack**: if any one template in a
pack fails to parse, fails to compile, or collides with an id that already
exists in the registry, the WHOLE pack is rejected — none of its templates
are registered, not even the ones that were fine on their own.

## Security note for anyone loading a pack

A template's `layout` body is JavaScript that runs directly in the browser
when drawcast draws with it. Only load a pack — from this repo or anywhere
else — that you trust. drawcast's UI shows this warning (and requires
confirmation) for any pack loaded from a custom URL; packs listed in this
repo's `index.json` are shown without that confirmation, precisely because
they're expected to have gone through review via a PR here.

## How to add a pack (PR)

1. Fork this repo.
2. Add `packs/<your_pack_id>.yaml` following the format above. Keep a pack
   focused (2–6 templates around one theme) — see `packs/showcase.yaml` for
   a minimal two-template example.
3. Add one entry to `index.json` (see the contract below) pointing `url` at
   the **raw** GitHub URL your pack file will have once merged, e.g.
   `https://raw.githubusercontent.com/hmelberg/drawcast-templates/main/packs/<your_pack_id>.yaml`.
4. Open a PR. Before merging, a maintainer loads your pack's raw URL in
   drawcast's "Add pack from URL…" to confirm it parses, registers, and every
   example renders lint-clean.

## `index.json` contract

`index.json` is a flat JSON array. Each entry:

```json
{
  "id": "your_pack_id",
  "title": "Human-readable title",
  "description": "One sentence shown in the Browse list.",
  "url": "https://raw.githubusercontent.com/hmelberg/drawcast-templates/main/packs/your_pack_id.yaml"
}
```

- `id` should match the pack's own `pack:` header id (not enforced by the
  format, but expected — mismatches are confusing).
- `url` must be an `https://` URL to the raw pack YAML (drawcast's fetcher
  refuses non-https URLs and anything over 500,000 characters).
