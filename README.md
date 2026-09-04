# renovate-config

Shared Renovate presets for Obitrain repos.

- `default.json5` — common base (npm-only, nightly `before 6am` Europe/Paris, semantic commits, dashboard).
- `lib.json5` — RN library policy (peerDeps manual, dev-tooling automerge, build backbone + RN core manual).

Usage in a library repo's `renovate.json5`:

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: ["github>Obitrain/renovate-config:lib"],
}
```

The Mend Renovate app needs read access to this repo (installed org-wide it just works).
Self-hosted runs (obiapp on GitLab) need `GITHUB_COM_TOKEN` to fetch `github>` presets.
Pin a revision with `github>Obitrain/renovate-config:lib#<tag>` if a config change must not apply instantly everywhere.
