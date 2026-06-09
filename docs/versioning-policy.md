# Versioning & Stability Policy

Codeman follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`),
managed via `@changesets/cli` (see the COM workflow in `CLAUDE.md`).

This document defines **what the version number actually promises** — i.e. which
surfaces are covered by SemVer and which are explicitly not. It exists because
"1.0" is a commitment to stability, and an undocumented public surface invites
incompatible client assumptions we would then be pressured to keep.

> **Status:** draft for the 1.0 cut. The central decision below — that the HTTP/SSE
> API is *not* SemVer-covered — should be confirmed by the maintainer before 1.0,
> since it determines whether a number of in-flight cleanups are "breaking."

## What SemVer covers (the public, stable surface)

A **MAJOR** bump is required to break any of these after 1.0:

1. **The CLI.** Command names, documented flags, and their behavior for
   `codeman <command>` (published to npm as `aicodeman`; invoked as `codeman`).
   This is the package's actual public entry point (`bin`).
   - *Note:* the npm package name vs. invoked command name (`aicodeman` vs
     `codeman`) is a known inconsistency to resolve **before** 1.0 — renaming
     either after 1.0 is itself a breaking change.
2. **The published `xterm-zerolag-input` library**, but on **its own version
   line** — it is versioned and released independently of the Codeman app. Its
   1.0 status is a separate decision; the Codeman app reaching 1.0 does *not*
   imply `xterm-zerolag-input` is 1.0.
3. **Documented environment variables** that configure deployment:
   `CODEMAN_PASSWORD`, `CODEMAN_USERNAME`, `CODEMAN_HOST`, `CODEMAN_PORT`,
   `CODEMAN_INSTANCE`, `CODEMAN_ALLOWED_HOSTS`, `CODEMAN_DATA_DIR`,
   `CODEMAN_TMUX_SOCKET`, and the `--host` / `--port` / `--https` CLI flags.
   Removing or changing the meaning of one of these is breaking.

## What SemVer does NOT cover (internal surfaces — may change in any release)

These may change in a **MINOR** (or even PATCH) release without a MAJOR bump:

1. **The HTTP API and SSE event registry** (`/api/...`, the ~120 SSE event
   types). This is an **internal protocol between the server and its own bundled
   web frontend**, not a published client API. There is no OpenAPI spec, no
   published client, and no versioned `/api/v1` namespace. If you script against
   these endpoints, **pin to an exact Codeman version** — they can change between
   minors (response shapes, status codes, event names). Standardizing the error
   envelope and HTTP status codes is explicitly reserved as a non-breaking
   internal change under this policy.
2. **The `~/.codeman/` state file formats** (`state.json`, `settings.json`,
   `mux-sessions.json`, etc.). We make a **best-effort** to migrate existing data
   forward (and have done so across renames), but the on-disk schema is not a
   stable contract — do not write tooling that depends on its exact shape.
3. **Internal TypeScript modules.** The npm package is CLI-only; `import`ing it
   programmatically is not supported (there is no stable library entry point).
4. **Experimental / opt-in features**, regardless of the app's version:
   Gesture Control (beta), Agent Teams
   (`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`), and anything labeled experimental
   in the UI or docs. These may change or be removed at any time.

## Deprecation policy

When we need to change a covered surface:

- Prefer **additive** changes (new flag/env var/command) over breaking ones.
- A covered surface slated for removal is **deprecated first** — it keeps working
  for at least one MINOR release with a runtime warning and a `CHANGELOG.md` note
  pointing to the replacement — then removed in the next MAJOR.
- Back-compat migration shims (e.g. the historical Claudeman→Codeman data/socket
  migration) are kept until a MAJOR boundary, then may be dropped.

## Pre-1.0 (`0.x`) caveat

Until 1.0 ships, **any release may contain breaking changes** per SemVer's `0.x`
allowance. The commitments above take effect at `1.0.0`.

## See also

- `CLAUDE.md` — the COM release workflow (changesets, version bump, deploy)
- `SECURITY.md` — security reporting and the supported-version policy
- `docs/security-architecture.md` — the full trust model
