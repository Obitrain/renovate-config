# renovate-config

Shared [Renovate](https://docs.renovatebot.com) presets for the Obitrain repos (GitHub + GitLab). Andarius and polarsen-io have their own `renovate-config` repo.
Plain JSON, no comments — Renovate fetches remote presets as `<name>.json` only
(no `.json5` fallback), so the rationale lives here instead.

## `default.json` — common base

- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended) + dependency dashboard + semantic commits (`chore(deps): ...`).
- All managers (npm, uv/pip, Docker, GitHub Actions, GitLab CI, …) except gradle (Android) / CocoaPods (iOS) /
  bundler: native platform upgrades stay manual.
- **Vulnerabilities**: [OSV](https://osv.dev) alerts on top of the platform's own; fix PRs skip the release-age
  buffer, open immediately and get the `security` label.
- **CI** (Actions / GitLab CI) non-major bumps: one grouped PR. **Non-npm majors**: Dependency Dashboard approval, `major-bump`.
- No `schedule`: the runner's nightly cron is the only time gate. GitHub starts it hours late
  (runs have landed ~5h late), so an in-config window like `before 6am` would never match.
- `minimumReleaseAge: 3 days`: yarn ≥ 4.18 quarantines releases younger than a day, so fresher
  proposals fail `yarn install`; also a supply-chain buffer.
- **Expo SDK patches**: keep the 3-day buffer, automerged on green CI.
- **Own packages** (`@obitrain/*`, `react-native-zstd`): 1 day (yarn's quarantine floor) — trusted, no buffer needed.
- **Node majors**: PR opened without Dashboard approval, so repos follow the latest LTS (Renovate's `node` versioning only offers LTS lines).
- `rangeStrategy: bump` (move the `^range`, keep the caret), 5 concurrent PRs, `dependencies` label.
- `rebaseWhen: behind-base-branch`: every PR re-runs CI on the latest main, so a fix landing on main
  (e.g. an Expo SDK patch) turns the other PRs green without manual rebases.

## `lib.json` — RN library policy (extends the base)

For [react-native-builder-bob](https://github.com/callstack/react-native-builder-bob)
library repos ([obiapp-ui](https://github.com/Obitrain/obiapp-ui),
[react-native-zstd](https://github.com/Andarius/react-native-zstd),
[obi-google-auth](https://github.com/Obitrain/obi-google-auth), ...).
Rules, in order:

- **peerDependencies + engines: hands off** — they're the compatibility contract; bump by hand.
- **dev tooling** (devDeps minor/patch): batched into one PR, automerged on green CI — excluding
  everything below, which must never ride along.
- **RN / React packages, in devDeps and the example app** (`react*`, `@types/react*`, `react-native-*`,
  `@react-native-community/*`, `@shopify/react-native-*`): no routine bumps — they track the Expo
  SDK / align-deps (expo-doctor and align-deps fail otherwise); majors still surface. Expo packages
  move through Renovate's own "expo monorepo" group.
- **Expo alignment**: any Expo bump runs `expo install --fix` in `example/` then `yarn dedupe` (post-upgrade task; expo-doctor also fails on duplicate native modules), so the
  PR also aligns the SDK-pinned packages the rules above never bump. Needs the runner's
  `RENOVATE_ALLOWED_COMMANDS`.
- **build backbone** (`react-native-builder-bob`, `turbo`): solo PR, manual — a bad bump breaks the build.
- **nitro** (`nitrogen` + `react-native-nitro-modules`): one PR together, manual — generated code must
  match the runtime, and 0.x minors are breaking. Inert in repos without these deps.
- **RN core** (`react-native`, `@react-native/*`, cli): fully ignored — platform upgrades are manual (rn-upgrade).
- **majors**: created only when approved on the Dependency Dashboard (except Node, see base), never automerged, labeled `major-bump`.

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
(`.github/workflows/renovate.yml`, daily 18:00 UTC + `workflow_dispatch`),
running [`renovatebot/github-action`](https://github.com/renovatebot/github-action)
against the explicit repo list — not the [Mend app](https://github.com/apps/renovate). It runs as the Renovate GitHub App: the `RENOVATE_APP_CLIENT_ID` variable and `RENOVATE_APP_PRIVATE_KEY` secret mint a 1h token per run. Repos without a config get an onboarding PR proposing the base
preset (switch RN libraries to `:lib`). GitHub suspends cron in repos inactive >60 days — re-enable from the
Actions tab when the reminder email arrives.

GitLab projects are autodiscovered by the scheduled
`renovate` job in [obitrain/renovate](https://gitlab.com/obitrain/renovate); obiapp keeps its own scheduled CI job. Self-hosted GitLab runs mint
`GITHUB_COM_TOKEN` from the same GitHub App to fetch `github>` presets. Pin a revision with
`github>Obitrain/renovate-config:lib#<tag>` if a config change must not apply
instantly everywhere. This repo stays public so cross-owner preset fetches
(e.g. Andarius/react-native-zstd) keep working.
