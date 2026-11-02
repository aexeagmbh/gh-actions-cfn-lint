# gh-actions-cfn-lint

This action runs cfn-lint. cfn-lint-ax is installed alongside it, but its rules only run once you
enable them (see below).


## Requirements

- A Linux or macOS runner. Windows runners are not supported.
- Python as `python` on the `PATH`, with the `venv` module (on Debian/Ubuntu: the `python3-venv`
  package), e.g. via `actions/setup-python`. cfn-lint declares the Python versions it supports in
  its package metadata (currently `>=3.10,<3.15`), and pip will not install it outside that range.
  Pin a supported minor version such as `python-version: "3.14"` rather than `3.x`, which picks up
  a new Python release before cfn-lint supports it.


## Versions

The action installs exact versions of cfn-lint, cfn-lint-ax and their dependencies, checked
against hashes, from `requirements.txt`. That file is compiled from the ranges in
`requirements.in`:

```sh
uv pip compile requirements.in --universal --python-version 3.10 --generate-hashes --output-file requirements.txt
```

`--universal` resolves for every Python from 3.10 on, pinning a package once per Python version
where one version can't serve them all. Compile against PyPI: if your uv config sets another
index, add `--no-config --default-index https://pypi.org/simple`. Another index can add md5
hashes, which pip refuses.

Dependabot updates `requirements.txt` weekly within the ranges in `requirements.in`, once a
release is 7 days old, and its pull requests merge by themselves when the tests pass. Repositories
using the action from `@main` get an update when it merges; repositories that pin the action to a
commit get it when they move the pin. Major versions of cfn-lint and cfn-lint-ax are capped in
`requirements.in`; adopting one means raising the cap and compiling again.


## Tests

`.github/workflows/test.yml` runs the action on the fixtures in `test/`: one appends the
cfn-lint-ax rules in the project's `.cfnlintrc`, the other in `~/.cfnlintrc`. Each lints a template
that must pass and one that must fail, as only the cfn-lint-ax rules reject it.


## Inputs

### `working-directory`

**Optional** Specifies the working directory where the commands are run.


## Enabling the cfn-lint-ax rules

The action runs `cfn-lint` without arguments, so cfn-lint reads its settings from `.cfnlintrc`
(or `.cfnlintrc.yaml` / `.cfnlintrc.yml`) in the directory given by the `working-directory` input
(default `./`). That file must list the templates to lint under `templates`. Without it, cfn-lint
reads a template from stdin, which is empty in CI, and fails with
`E1001 'Resources' is a required property`.

cfn-lint does not load the cfn-lint-ax rules by itself; add them under `append_rules`:

```yaml
templates:
  - "**/*.template.yaml"
append_rules:
  - cfn_lint_ax.rules
  - cfn_lint_ax.serverless_rules
```

The rules are listed in [aexeagmbh/cfn-lint-rules](https://github.com/aexeagmbh/cfn-lint-rules).


## Example usage

```yaml
uses: aexeagmbh/gh-actions-cfn-lint@main
with:
  working-directory: ./foo
```
