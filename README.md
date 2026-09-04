# renovate-config

Shared [Renovate](https://docs.renovatebot.com) presets for Obitrain repos.
Plain JSON, no comments — Renovate fetches remote presets as `<name>.json` only
(no `.json5` fallback), so the rationale lives here instead.

## `default.json` — common base

- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended) + dependency dashboard + semantic commits (`chore(deps): ...`).
- npm manager only — never touch gradle (Android) / CocoaPods (iOS) / bundler / CI manifests.
- Nightly window: `before 6am` Europe/Paris — off-hours, spares the self-hosted M2 runners.
- `rangeStrategy: bump` (move the `^range`, keep the caret), 5 concurrent PRs, `dependencies` label.

## `lib.json` — RN library policy (extends the base)

For [react-native-builder-bob](https://github.com/callstack/react-native-builder-bob)
library repos ([obiapp-ui](https://github.com/Obitrain/obiapp-ui),
[react-native-zstd](https://github.com/Andarius/react-native-zstd),
[obi-google-auth](https://github.com/Obitrain/obi-google-auth), ...).
Rules, in order:

- **peerDependencies: hands off** — they're the RN/react compatibility contract; bump by hand.
- **dev tooling** (devDeps minor/patch): batched into one PR, automerged on green CI.
- **build backbone** (`react-native-builder-bob`, `nitrogen`, `react-native-nitro-modules`, `turbo`): manual — a bad bump breaks the package build/codegen; nitrogen must move in lockstep with nitro-modules. Inert in repos without these deps.
- **RN core** (`react-native`, `@react-native/*`, cli): fully ignored — platform upgrades are manual (rn-upgrade).
- **majors**: always solo + manual, labeled `major-bump`.

Repos with diverging policy ([obi-chart](https://github.com/Obitrain/obi-chart): weekly cadence, Expo-bundled natives,
align-deps ownership where RN-core majors must still surface) extend only the
base and keep their rules inline.

## Usage

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: ["github>Obitrain/renovate-config:lib"],
}
```

## Runner

The GitHub repos are driven by this repo's own cron workflow
(`.github/workflows/renovate.yml`, nightly 03:00 UTC + `workflow_dispatch`),
running [`renovatebot/github-action`](https://github.com/renovatebot/github-action)
against the explicit repo list — not the [Mend app](https://github.com/apps/renovate). It needs the `RENOVATE_TOKEN` Actions secret: a **classic** PAT with
`repo` scope (classic spans both `Andarius` and `Obitrain`; fine-grained PATs
are per-owner). Repos without a config get an onboarding PR proposing the lib
preset. GitHub suspends cron in repos inactive >60 days — re-enable from the
Actions tab when the reminder email arrives.

obiapp (GitLab) keeps its own scheduled CI job. Self-hosted GitLab runs need
`GITHUB_COM_TOKEN` to fetch `github>` presets. Pin a revision with
`github>Obitrain/renovate-config:lib#<tag>` if a config change must not apply
instantly everywhere. This repo stays public so cross-owner preset fetches
(e.g. Andarius/react-native-zstd) keep working.
