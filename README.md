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


## Inputs

### `working-directory`

**Optional** Specifies the working directory where the commands are run.


## Enabling the cfn-lint-ax rules

The action runs cfn-lint with the settings in `.cfnlintrc` (or `.cfnlintrc.yaml` /
`.cfnlintrc.yml`) in the directory given by the `working-directory` input (default `./`), passed
as `--config-file`. That file must list the templates to lint under `templates`. Without it,
cfn-lint reads a template from stdin, which is empty in CI, and fails with
`E1001 'Resources' is a required property`.

The file is passed explicitly because cfn-lint 1.59.0 and later refuse `append_rules` in a
project config they discover on their own (`'append_rules' cannot be set in …/.cfnlintrc`). To run
cfn-lint the same way locally, pass it too: `cfn-lint --config-file .cfnlintrc`.

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
