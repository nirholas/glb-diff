# @three-ws/glb-diff

<!-- three.ws:badges -->
[![GitHub stars](https://img.shields.io/github/stars/nirholas/glb-diff?style=flat&logo=github)](https://github.com/nirholas/glb-diff/stargazers) [![License](https://img.shields.io/github/license/nirholas/glb-diff?style=flat)](https://github.com/nirholas/glb-diff/blob/HEAD/LICENSE) [![Last commit](https://img.shields.io/github/last-commit/nirholas/glb-diff?style=flat)](https://github.com/nirholas/glb-diff/commits) [![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat)](https://github.com/nirholas/glb-diff/pulls) [![AI agent friendly](https://img.shields.io/badge/AI%20agents-AGENTS.md%20%2B%20llms.txt-6d5dfc?style=flat)](https://github.com/nirholas/glb-diff/blob/HEAD/AGENTS.md)
<!-- /three.ws:badges -->


**Structural diff for glTF/GLB.** Tells you what actually changed between two 3D
models: geometry, materials, textures, skeletons, and animations, with git-style
rename and move detection.

A binary diff of two GLB files tells you the bytes differ, which you already
knew. This tells you the thing you need to know before you ship: whether a
consumer of that model is about to break.

## Install

```bash
npm install @three-ws/glb-diff
```

## Library

```js
import { readFile } from 'node:fs/promises';
import { diffModels, formatText } from '@three-ws/glb-diff';

const before = await readFile('avatar.v1.glb');
const after = await readFile('avatar.v2.glb');

const changes = await diffModels(before, after, { nameA: 'v1', nameB: 'v2' });
console.log(formatText(changes));

if (changes.severity === 'breaking') process.exit(1);
```

Isomorphic: it takes bytes, so the same call works in Node, in a worker, in a
serverless handler, or in a browser.

## CLI

```bash
glb-diff before.glb after.glb
glb-diff before.glb https://example.com/after.glb --markdown
glb-diff base.glb candidate.glb --fail-on major
```

| Option | Meaning |
| --- | --- |
| `--json` | The full change set as JSON |
| `--markdown` | A Markdown report, sized for a pull-request comment |
| `--fail-on <level>` | Exit 1 once severity reaches this level |
| `--verbose` | Include unchanged totals in the table |

Exit codes are the contract a CI job depends on, so they are deliberate:

| Code | Meaning |
| --- | --- |
| `0` | The diff ran and stayed below `--fail-on` |
| `1` | The diff ran and the severity reached `--fail-on` |
| `2` | The tool could not run: bad arguments, unreadable input, unparseable model |

A tool that returns `1` for both "your model regressed" and "I could not open the
file" is useless in a pipeline, which is why `2` exists.

## Severity

Every change set carries one severity, the worst of everything it found:

| Level | Meaning |
| --- | --- |
| `none` | The two models are structurally identical. |
| `cosmetic` | Only metadata changed. Nothing a renderer or a clip can observe. |
| `minor` | Appearance changed. The model still loads, animates, and keeps its shape. |
| `major` | Geometry or hierarchy changed. Anything positioned against this model should be re-checked. |
| `breaking` | Something a consumer references by name is gone. Clips, attachments, or materials bound to it will fail. |

That ladder is what makes `--fail-on` useful. A texture swap should not fail a
build; a deleted bone that every animation clip targets should.

## API

| Export | What it does |
| --- | --- |
| `diffModels(bytesA, bytesB, opts?)` | Bytes in, change set out. The one call most callers need. |
| `describeModel(bytes)` / `describeDocument(doc)` | The structural description a diff is computed from. |
| `diffDescriptions(a, b)` | Diff two descriptions you already have, skipping the parse. |
| `formatText(changes)` / `formatMarkdown(changes)` | Render a change set for a terminal or a PR comment. |
| `readDocument(bytes)` / `isGLB(bytes)` | Parse helpers over `@gltf-transform/core`. |
| `SEVERITIES`, `SEVERITY_MEANING`, `atLeast()`, `maxSeverity()`, `severityRank()` | The severity ladder as data. |
| `matchEntries()`, `jaccard()`, `ratio()` | The similarity matcher behind rename and move detection. |

`DESCRIPTION_VERSION` and `CHANGESET_VERSION` are exported so a cache or a stored
report can tell whether it was produced by a compatible version.

## Related

- [`@three-ws/render`](../render): rasterize a GLB to PNG with no GPU, for a
  visual check alongside the structural one.
- [`STRUCTURE.md`](../../STRUCTURE.md): where every three.ws surface lives.

## License

Apache-2.0

<!-- three.ws:growth -->
## Support the project

If glb-diff saves you time, **[star it on GitHub](https://github.com/nirholas/glb-diff)**. Stars are how other developers and AI agents find the repositories worth trusting, and they cost you one click.

Know someone who would use it? [Post on X](https://twitter.com/intent/tweet?text=glb-diff%3A%20Structural%20diff%20for%20glTF%2FGLB&url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fglb-diff) · [Share on Bluesky](https://bsky.app/intent/compose?text=glb-diff%3A%20Structural%20diff%20for%20glTF%2FGLB%20https%3A%2F%2Fgithub.com%2Fnirholas%2Fglb-diff) · [Share on LinkedIn](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fglb-diff) · [Submit to Hacker News](https://news.ycombinator.com/submitlink?u=https%3A%2F%2Fgithub.com%2Fnirholas%2Fglb-diff&t=glb-diff%3A%20Structural%20diff%20for%20glTF%2FGLB) · [Share on Reddit](https://www.reddit.com/submit?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Fglb-diff&title=glb-diff%3A%20Structural%20diff%20for%20glTF%2FGLB)

## Built for AI agents too

Coding agents and LLM tooling can read this repo directly: [AGENTS.md](./AGENTS.md), [llms.txt](./llms.txt), [llms-full.txt](./llms-full.txt). Point an agent at `https://github.com/nirholas/glb-diff` and it has the context it needs.

## More from the same author

- [All repositories by nirholas](https://github.com/nirholas/nirholas#readme): the full catalog, grouped by topic
- [three.ws](https://three.ws): the platform for 3D AI agents with Solana wallets, a skill marketplace and x402 payments
- Questions or ideas: [open an issue](https://github.com/nirholas/glb-diff/issues) or [start a discussion](https://github.com/nirholas/glb-diff/discussions)

## Contributors

[![Contributors](https://contrib.rocks/image?repo=nirholas/glb-diff)](https://github.com/nirholas/glb-diff/graphs/contributors)

## Star history

[![Star History Chart](https://api.star-history.com/svg?repos=nirholas/glb-diff&type=Date)](https://www.star-history.com/#nirholas/glb-diff&Date)

<!-- /three.ws:growth -->

<!-- three.ws:ecosystem -->
## Part of three.ws

[three.ws](https://three.ws) is a platform for 3D AI agents with Solana wallets: avatars, a skill marketplace, x402 payments and more than seventy MCP servers. `@three-ws/glb-diff` is one package from it.

- Documentation site: https://nirholas.github.io/glb-diff/
- npm: https://www.npmjs.com/package/@three-ws/glb-diff
- Canonical source: https://github.com/nirholas/three.ws (this repository is a generated mirror of `packages/glb-diff`)
- Agent-readable summary: [llms.txt](./llms.txt) and [AGENTS.md](./AGENTS.md)
- Issues and ideas: https://github.com/nirholas/glb-diff/issues
<!-- /three.ws:ecosystem -->
