# agent-skills

Eleven Skills for coding agents, published by Floom. Ten are edited copies from [Federico de Ponte's](https://github.com/federicodeponte) working set. The eleventh, `get-started-with-cyborg-mcp`, was written for this repository by Floom and is about Floom's own Skill catalog.

A Skill is a folder with a `SKILL.md` at its root: a short front matter block naming the Skill and saying when to invoke it, then the instructions themselves. Agents that support Skills read the front matter to decide when a Skill applies, and the body once it does. Some of these carry scripts the instructions call.

## Ten of these are derived copies, and each one says how it was derived

Ten of the eleven started in Federico's own working set. The original was read, never modified. What is published is a copy, edited so that it is useful to a stranger rather than only to the person who wrote it.

Each of those ten folders carries a `DERIVATION.json` recording that edit in full: the source location, every file copied, every file left behind, the SHA-256 of each, and the licence that was added. A record cannot contain its own hash, so `DERIVATION.json` is excluded from the file list it describes.

The point of publishing the record alongside the copy is that you do not have to take the word "derived" on trust. You can read exactly what changed.

## Licence

Floom's contributions are Apache-2.0. The root `LICENSE` carries the full terms, and each derived bundle carries its own copy. The existing nine derived bundles applied that licence on 2026-09-07; `openpaper` applies it on 2026-09-09. The text is upstream Apache-2.0 unmodified except for the appendix copyright line, which is what the Apache appendix instructs a licensor to fill in. `openpaper` also retains the MIT licence and copyright notice for the OpenDraft-derived material in `THIRD_PARTY_NOTICES.md`.

You may use, modify and redistribute these, subject to the licence's attribution requirement. The grant applies from the version it appears on and cannot be withdrawn from a version already fetched.

Before their recorded licence dates these copies carried no licence file at all, which under default copyright meant a stranger could read one and not modify or redistribute it. That was the wrong default for work meant to be installed by other people, and it is the one that applies when nobody chooses.

## What is not claimed

No evaluation has been run against any of these, and no quality or safety state is asserted. Ten are working instructions, published because they were useful in practice, not because they passed a general quality bar. The eleventh is documentation of a product Floom builds, and it names, by version, which agents it was actually run against and which it was not.

Several call out to tools that must already be on your machine: `generate-image` drives the Codex CLI, `linkedin-media-prep` and `strip-image-ai-metadata` use ffmpeg and Python imaging libraries, and `security-audit-checklist` bundles three Python scanners. Read a Skill's instructions and its scripts before you run it, the same as any other code you install.

## The eleven

| Skill | What it does |
| --- | --- |
| `cli-ux-review` | Scores a command-line tool against a fixed rubric and writes the before/after fix for each failure. |
| `get-started-with-cyborg-mcp` | Wires Floom's public Skill catalog into an agent through the `cyborg-mcp` MCP server, and installs a first Skill. Written here, not derived. |
| `generate-image` | Generates images through the Codex CLI, billed to a ChatGPT subscription rather than a per-image API key. |
| `http-error-triage` | Separates a real credential problem from a CDN block, a wrong endpoint or a signature ban, before anyone concludes "the key is dead". |
| `linkedin-media-prep` | Converts, crops and compresses images and video to what LinkedIn actually accepts. |
| `openpaper` | Turns one topic into a research-paper draft with DOI-backed source lookup and an explicit citation-integrity gate. |
| `security-audit-checklist` | Audits app code, cloud config, containers, CI and IaC, with three bundled scanners. |
| `shadcn-first` | Builds UI from shadcn blocks and components instead of hand-written markup. |
| `strip-image-ai-metadata` | Strips C2PA and AI-generation metadata so platforms stop labelling an image. |
| `top-down-comms` | Structures a client-facing artifact the way MBB consultants do: governing thought first. |
| `workplan` | Creates, updates and closes a work plan so multi-step work survives losing context. |

## Installing one

Each Skill is a plain folder. Copy it into wherever your agent reads Skills from, or install it with a tool that understands the format.

```
git clone https://github.com/floomhq/agent-skills
cp -r agent-skills/workplan ~/.claude/skills/workplan
```
