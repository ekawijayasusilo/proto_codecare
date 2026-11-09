# proto_codecare

Prototype for Codecare, a collection of AI workflows to support end-to-end SDLC process in GitHub repo.

This repo hosts **reusable GitHub workflows** and **copy-paste stubs** for Proto CodeCare client repos, plus the
central **Renovate runner**. Renovate presets, the runner's global config and scripts live in
[`proto_codecare_scripts`](https://github.com/ekawijayasusilo/proto_codecare_scripts).

## Stub catalogue

Copy a stub from [`stubs/`](stubs) into the client repo's `.github/workflows/` (or `.github/` for `dependabot.yaml`)
and edit only the inputs listed.

| Stub | Goal | Use when | Inputs to edit | Secrets |
|---|---|---|---|---|
| `flutter-ci.yml` | Format, analyze, test, coverage, lockfile check | Every Flutter repo | `flutter_version_file`, coverage options | `CODECOV_TOKEN` (optional) |
| `flutter-android-build.yml` | Prove the Android project builds with the pinned Flutter | Flutter apps with `android/` | `build_args` (flavor, entrypoint) | none |
| `semantic-pull-request.yml` | Conventional PR titles (+ `vulnerability`) | Every repo using squash merges with PR titles | none | none |
| `dependabot.yaml` | Dependabot security PRs only | Every repo with pub or GitHub Actions deps | ecosystems/directories | none |

- `flutter_version_file` needs an **exact** `environment.flutter` version in `pubspec.yaml`. Set it or
  `flutter_version`, not both.
- Don't make `flutter-android-build` a required status check: its `paths` filter skips most PRs, and a required check
  that never runs blocks the PR.

## Dependency updates

```txt
proto_codecare_scripts ──globalExtends runner#vX──► proto_codecare/.github/workflows/renovate.yml
   │                                                  │ scheduled, GitHub App token (secrets live ONLY here)
   │ extends default + flutter.json5#vX               ▼ opens PRs in
   ▼                                               client repo
client .github/renovate.json5                        .github/workflows/*.yml stubs ──uses──► proto_codecare reusables @vX
                                                     .github/dependabot.yaml (stub copy)       (no secrets needed)
```

| | Renovate | Dependabot |
|---|---|---|
| Handles | Routine version updates (pub, GitHub Actions, Flutter SDK + Android tooling) | Security updates for pub (Renovate can't fix pub vulnerabilities) |
| Runs | Central runner in this repo | GitHub, per client repo |
| Config | Presets in `proto_codecare_scripts` | `stubs/dependabot.yaml` copy |

**Opt a client repo in:**

1. Install the Renovate GitHub App on the repo.
2. Add `.github/renovate.json5` from `proto_codecare_scripts`'
   [example](https://github.com/ekawijayasusilo/proto_codecare_scripts/blob/main/version-control/renovate-configs/examples/flutter-app.renovate.json5).
   Without it the runner skips the repo.
3. Copy the stubs.
4. Turn on Dependabot alerts and security updates (`setup-github-repo.sh --dependabot`).

## Renovate runner

[`.github/workflows/renovate.yml`](.github/workflows/renovate.yml) runs daily at 03:17 Asia/Jakarta and on manual
dispatch. One run covers every repo the GitHub App is installed on.

- **Auth:** a GitHub App token. `RENOVATE_APP_ID` (variable) and `RENOVATE_APP_PRIVATE_KEY` (secret) exist **only in
  this repo**; client repos need no secrets.
- **Global config:** `RENOVATE_GLOBAL_EXTENDS` loads `proto_codecare_scripts`' `runner` preset at an exact tag
  (autodiscover, `requireConfig`, `allowedCommands`).
- **Keeping that tag current:** no built-in Renovate manager reads env vars, so a regex manager in
  [`renovate.json`](renovate.json) bumps it. Otherwise new `postUpgradeTasks` commands in client presets would be
  rejected by an outdated `allowedCommands`.

> **Warning:** Actions logs of a public repo are public. The runner log lists the repos it processes and their
> dependencies. Move the runner to a private repo before adding private client repos.

## Versioning

- Releases and `vX.Y.Z` tags come from release-please. Each release PR rewrites the `uses:` tag in the stubs (lines
  marked `# x-release-please-version`), so a freshly copied stub points at the latest release.
- In client repos Renovate bumps the tag; patches automerge.
- Tags are protected and never moved.
- Release rules: a change consumers notice is `feat:` / `fix:`; an incompatible input change is `feat!:`; docs-only is
  `docs:`.
