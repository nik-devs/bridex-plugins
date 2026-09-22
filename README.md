# bridex-plugins

The plugin registry for [Bridex](https://bridex.app) instances — the same idea
as a Linux package source: one `index.yaml` lists every plugin with its
version and a pinned commit, and instances read only this file to render
their catalog and detect updates.

## How an instance uses it

1. The registry URL is configured on the instance (this repo is the default).
2. The dashboard's Plugins → Catalog tab renders the entries below.
3. **Install** clones the plugin repository at the *pinned commit* into the
   instance volume (`$BRIDEX_HOME/plugins/<name>/`) and installs its
   dependencies into a per-plugin `node_modules`.
4. **Updates** are detected by comparing the installed version against
   `index.yaml` — no plugin repositories are contacted for the check.

## Entry format

```yaml
- name: hello                # directory + plugin.yaml name (must match)
  kind: tool                 # tool | channel | integration
  description: One line shown on the catalog card
  version: 0.1.0             # semver; drives update detection
  repo: https://github.com/nik-devs/bridex-plugin-hello
  ref: main
  commit: <full sha>         # the exact commit an install checks out
  permissions:               # shown on the card BEFORE install
    network: false           # outbound network access
    llm: false               # may run completions via the calling agent's model
```

## Adding a plugin

Open a PR that adds an entry to `index.yaml`. The plugin repository must
contain `plugin.yaml` (name, version, kind, description, entry, optional
`requires_env` and `config_schema` — the latter renders the plugin's settings
form in the dashboard) and an ES module exporting `activate(ctx)` — see
[bridex-plugin-hello](https://github.com/nik-devs/bridex-plugin-hello) for the
minimal working example.

Releasing a new version = commit in the plugin repo, then a PR here bumping
`version` and `commit`.
