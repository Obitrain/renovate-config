# renovate-config

Shared Renovate presets for Obitrain repos. Plain JSON, no comments — Renovate
fetches remote presets as `<name>.json` only (no `.json5` fallback), so the
rationale lives here instead.

## `default.json` — common base

- `config:recommended` + dependency dashboard + semantic commits (`chore(deps): ...`).
- npm manager only — never touch gradle (Android) / CocoaPods (iOS) / bundler / CI manifests.
- Nightly window: `before 6am` Europe/Paris — off-hours, spares the self-hosted M2 runners.
- `rangeStrategy: bump` (move the `^range`, keep the caret), 5 concurrent PRs, `dependencies` label.

## `lib.json` — RN library policy (extends the base)

For react-native-builder-bob library repos (obiapp-ui, react-native-zstd, obi-google-auth, ...).
Rules, in order:

- **peerDependencies: hands off** — they're the RN/react compatibility contract; bump by hand.
- **dev tooling** (devDeps minor/patch): batched into one PR, automerged on green CI.
- **build backbone** (`react-native-builder-bob`, `nitrogen`, `react-native-nitro-modules`, `turbo`): manual — a bad bump breaks the package build/codegen; nitrogen must move in lockstep with nitro-modules. Inert in repos without these deps.
- **RN core** (`react-native`, `@react-native/*`, cli): fully ignored — platform upgrades are manual (rn-upgrade).
- **majors**: always solo + manual, labeled `major-bump`.

Repos with diverging policy (obi-chart: weekly cadence, Expo-bundled natives,
align-deps ownership where RN-core majors must still surface) extend only the
base and keep their rules inline.

## Usage

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: ["github>Obitrain/renovate-config:lib"],
}
```

The Mend Renovate app needs read access to this repo (public, so cross-owner
consumers like Andarius/react-native-zstd work too). Self-hosted runs (obiapp on
GitLab) need `GITHUB_COM_TOKEN` to fetch `github>` presets. Pin a revision with
`github>Obitrain/renovate-config:lib#<tag>` if a config change must not apply
instantly everywhere.
