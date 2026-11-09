# proto_codecare

Prototype for Codecare, a collection of AI workflows to support end-to-end SDLC process in GitHub repo.

This repo hosts the central **Renovate runner** for Proto CodeCare client repos, and will host reusable GitHub
workflows. Renovate presets, the runner's global config and scripts live in
[`proto_codecare_scripts`](https://github.com/ekawijayasusilo/proto_codecare_scripts).

## Dependency updates

```txt
proto_codecare_scripts ──globalExtends runner#vX──► proto_codecare/.github/workflows/renovate.yml
   │                                                  │ scheduled, GitHub App token (secrets live ONLY here)
   │ extends default + flutter.json5#vX               ▼ opens PRs in
   ▼                                               client repo
client .github/renovate.json5                        .github/dependabot.yaml (stub copy, security only)
```

| | Renovate | Dependabot |
|---|---|---|
| Handles | Routine version updates (pub, GitHub Actions, Flutter SDK + Android tooling) | Security updates for pub (Renovate can't fix pub vulnerabilities) |
| Runs | Central runner in this repo | GitHub, per client repo |
| Config | Presets in `proto_codecare_scripts` | Copy of [`stubs/dependabot.yaml`](stubs/dependabot.yaml) |

**Opt a client repo in:**

1. Install the Renovate GitHub App on the repo.
2. Add `.github/renovate.json5` from `proto_codecare_scripts`'
   [example](https://github.com/ekawijayasusilo/proto_codecare_scripts/blob/main/version-control/renovate-configs/examples/flutter-app.renovate.json5).
   Without it the runner skips the repo.
3. Copy `stubs/dependabot.yaml` to `.github/dependabot.yaml` and turn on Dependabot alerts and security updates
   (`setup-github-repo.sh --dependabot`).

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

## Contributing

- Every PR runs [actionlint](https://github.com/rhysd/actionlint) on `.github/workflows/` (`lint.yml`).
- Releases and `vX.Y.Z` tags come from release-please. Tags are protected and never moved.
- Release rules: a change consumers notice is `feat:` / `fix:`; an incompatible change is `feat!:`; docs-only is
  `docs:`.
