**中文** · [README.md](./README.md)

<!-- badges -->
[![Node](https://img.shields.io/badge/Node.js-≥14-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Zero-Deps](https://img.shields.io/badge/dependencies-zero-0f9d58?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![License](https://img.shields.io/github/license/huanweide/pkgdoctor)](LICENSE)
<!-- /badges -->

# pkgdoctor

A zero-dependency, single-file CLI that checks the hygiene of your `package.json` — its metadata smells and dependency-declaration discipline — and reports a 0–100 health score with CI-ready exit codes.

It reads only your own `package.json`. It never touches the network or git, never executes your code. One file, zero dependencies, deterministic and reproducible.

## Why it exists

`package.json` is the face of a Node project, yet many repos cut corners on metadata: missing `license` (not legally reusable), undeclared `engines.node` (breaks on incompatible Node), dependencies pinned to `"*"` / `"latest"` (version drift, non-reproducible builds), no `test` script (CI has nothing to run).

Existing tools each leave a blind spot:

- `npm audit` checks **vulnerabilities** and needs the registry.
- `depcheck` checks **dead dependencies** and needs installation.
- `devdoctor` checks **dependency bloat + license + cycles + plaintext secrets**.
- `licguard` checks the **license of every package in node_modules** (others' code).
- `repodoctor` checks **repo engineering hygiene** (incl. "package.json without a lockfile").

`pkgdoctor` fills the gap none of them cover: the **configuration-spec layer of your own `package.json`** — who you are (name/author/repository), release compliance (license/version), reproducibility (dependency-range discipline / lockfile version), verifiability (test script), discoverability (description/keywords/type). Orthogonal, no overlap.

## Who should use it / Who shouldn't

- **Use it if** you maintain a public npm package, just inherited an old repo and want a quick configuration check-up of its `package.json`, or want to add a "config-spec gate" to CI.
- **Skip it if** you ship a pure front-end SPA that never publishes to npm and doesn't care about `license` / `engines`, or you already run a full Renovate + strict release pipeline with zero tolerance for deviation (it still works as an independent check, but the marginal value is small).

## Install / Run

No install needed — run the single file directly (Node ≥ 14):

```bash
node index.js .
```

Or link it globally as a command:

```bash
npm link        # then use the pkgdoctor command
pkgdoctor . --fail-on-high --min-score 80
```

## Usage

```bash
pkgdoctor [directory or package.json path] [options]
```

- No target → scans `package.json` in the current directory.
- A directory → scans that directory's `package.json`.
- A file path → scans that file directly.

Options:

| Option | Effect |
| --- | --- |
| `--json` | Output JSON (with `score` / `counts` / `issues`) for CI parsing |
| `--quiet` | Show only the CI gate result |
| `--fail-on-high` | Fail (exit 2) if any HIGH issue exists |
| `--max-high N` | Fail if more than N HIGH issues |
| `--max-medium N` | Fail if more than N MEDIUM issues |
| `--max-issues N` | Fail if total issues exceed N |
| `--min-score N` | Fail if health score is below N |
| `-V, --version` | Show version |
| `-h, --help` | Show help |

## Rules

| Severity | Rule | Meaning |
| --- | --- | --- |
| HIGH | `missing-name` | Missing `name` field |
| HIGH | `invalid-version` | `version` is not valid semver (should be `x.y.z`) |
| HIGH | `missing-license` | A public (non-`private`) package lacks a `license` field |
| MEDIUM | `wildcard-dependency` | Dependency pinned to `"*"` / `"x"` / `"latest"` etc. (drift risk) |
| MEDIUM | `missing-engines` | Missing `engines.node` (no declared Node range) |
| MEDIUM | `missing-repository` | Missing `repository` (hurts discoverability & contribution) |
| MEDIUM | `missing-test-script` | `scripts` lacks `test` (CI can't verify automatically) |
| MEDIUM | `missing-description` | Missing `description` (hurts npm discovery) |
| LOW | `missing-author` | Missing `author` |
| LOW | `missing-type` | Missing `type` (ESM/CJS ambiguity, defaults to commonjs) |
| LOW | `missing-keywords` | Missing `keywords` (hurts npm discovery) |
| LOW | `lockfile-outdated` | `package-lock.json` has an old `lockfileVersion` (< 2) |

> Wildcard exemptions: `workspace:*`, `file:` / `link:` / `git+https:` / `npm:` and other non-registry dependencies are not falsely flagged.

## Health score

Out of 100, points are deducted per issue weight and floored at 0:

- HIGH each −12
- MEDIUM each −6
- LOW each −3

Grades: `A`(≥90) `B`(≥75) `C`(≥60) `D`(≥40) `E`(<40).

## CI gate

Use it as a hard gate in GitHub Actions:

```yaml
name: pkg-health
on: [push, pull_request]
jobs:
  lint-pkg:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npx -y huanweide/pkgdoctor . --fail-on-high --min-score 80
```

Local pre-commit:

```bash
pkgdoctor . --fail-on-high --max-medium 0
```

## Zero false-positive discipline

- Pure static parsing — never runs your code, no network, no shell; no injection surface, no side effects.
- Legitimate wildcard ranges are exempted (`workspace:` / `file:` / `link:` / `git:` / `npm:`) so monorepos and local deps aren't falsely flagged.
- Private packages (`private: true`) aren't required to have a `license`, matching npm semantics.
- The tool's own `package.json` satisfies every rule and scores 100 (dogfooding, zero debt).

## Related tools

`pkgdoctor` is the configuration layer of the "engineering-health family", orthogonal to:

- [devdoctor](https://github.com/huanweide/devdoctor) (dependency bloat + license + cycles + plaintext secrets)
- [licguard](https://github.com/huanweide/licguard) (per-package license compliance in node_modules)
- [repodoctor](https://github.com/huanweide/repodoctor) (repo engineering hygiene)
- the source-layer set (testlite / debtlens / a11ydoctor / awaitscan / cycscan / secscan / dupscan / typedoctor)
- the git-layer set (docdoctor / commitdoctor / reldoctor / repodoctor)

## Roadmap

The rule set is stable. Planned additions: a `--fix` suggestion mode and configurable severity weights. No breaking changes are expected.

## Contributing

Bug reports and pull requests are welcome. The CLI is intentionally single-file and zero-dependency — keep it that way. It dogfoods itself: its own `package.json` must score 100.

## ⭐ Star

If pkgdoctor ever saved you from a "missing license / version drift" release incident, a Star ⭐ helps others find it: https://github.com/huanweide/pkgdoctor

## Author

[huanweide](https://github.com/huanweide)

## License

MIT
