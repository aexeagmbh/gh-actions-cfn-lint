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

Dependabot updates `requirements.txt` daily, once a release is 7 days old (cfn-lint-ax, our own
package, without waiting), and its pull requests merge by themselves when the tests pass. It stays
within the ranges in `requirements.in` and never changes them. Repositories using the action from
`@main` get an update when it merges; repositories that pin the action to a commit get it when
they move the pin.

`requirements.in` caps the major versions of cfn-lint and cfn-lint-ax, and holds cfn-lint below
1.59 for now (see [Enabling the cfn-lint-ax rules](#enabling-the-cfn-lint-ax-rules)). Moving a cap
means editing `requirements.in` and compiling again.


## Tests

`.github/workflows/test.yml` runs the action on every Python from 3.10 to 3.14, with two fixtures
in `test/`: one appends the cfn-lint-ax rules in the project's `.cfnlintrc`, the other in
`~/.cfnlintrc`. Each lints a template that must pass, then two that must fail, each breaking a rule
from only one of the two cfn-lint-ax packs.


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

cfn-lint 1.59.0 and later refuse `append_rules` in a `.cfnlintrc` they find on their own
(`'append_rules' cannot be set in …/.cfnlintrc`), so `requirements.in` holds the action at cfn-lint
1.58 until it handles that. To run cfn-lint 1.59 or later locally, put `append_rules` in
`~/.cfnlintrc` instead, or pass `--append-rules` on the command line.

The rules are listed in [aexeagmbh/cfn-lint-rules](https://github.com/aexeagmbh/cfn-lint-rules).


## Example usage

```yaml
uses: aexeagmbh/gh-actions-cfn-lint@main
with:
  working-directory: ./foo
```
