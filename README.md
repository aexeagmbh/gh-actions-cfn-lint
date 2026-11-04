# gh-actions-cfn-lint

This action runs cfn-lint with the cfn-lint-ax rules enabled (see below).


## Requirements

- A Linux or macOS runner. Windows runners are not supported.
- Python as `python` on the `PATH`, with the `venv` module (on Debian/Ubuntu: the `python3-venv`
  package), e.g. via `actions/setup-python`. cfn-lint declares the Python versions it supports in
  its package metadata (currently `>=3.10,<3.16`), and pip will not install it outside that range.
  Pin a supported minor version such as `python-version: "3.14"` rather than `3.x`, which picks up
  a new Python release before cfn-lint supports it. The action's tests cover Python 3.10 to 3.15
  (see [Tests](#tests)).


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

`requirements.in` caps the major versions of cfn-lint and cfn-lint-ax. Moving a cap means editing
`requirements.in` and compiling again.


## Tests

`.github/workflows/test.yml` runs the action on every Python from 3.10 to 3.15, with two fixtures
in `test/`. `templates-only` is the supported setup: a `.cfnlintrc` with only `templates`.
`user-config` has `append_rules` in `~/.cfnlintrc` next to the action's flag, which must not break
the run. Each lints a template that must pass, then two that must fail, each breaking a rule from
only one of the two cfn-lint-ax packs.


## Inputs

### `working-directory`

**Optional** Specifies the working directory where the commands are run.


## Configuring cfn-lint

The action passes nothing but the rule packs (see below) to `cfn-lint`; everything else cfn-lint
reads from `.cfnlintrc` (or `.cfnlintrc.yaml` / `.cfnlintrc.yml`) in the directory given by the
`working-directory` input (default `./`). That file must list the templates to lint under
`templates`. Without it, cfn-lint reads a template from stdin, which is empty in CI, and fails with
`E1001 'Resources' is a required property`.

The action enables both cfn-lint-ax packs itself, with
`cfn-lint --append-rules cfn_lint_ax.rules cfn_lint_ax.serverless_rules`. Your `.cfnlintrc` lists
`templates` and any other settings, but no `append_rules`:

```yaml
templates:
  - "**/*.template.yaml"
```

Without `merge_configs`, the command-line `--append-rules` replaces `append_rules` from config
files, so you can't add rule modules that way. With `merge_configs: true`, a `~/.cfnlintrc` that
lists the packs fails with `DuplicateRuleError`.

cfn-lint 1.59.0 and later refuse `append_rules` and `output_file` in the project's `.cfnlintrc`, the
one they find in the working directory (`'append_rules' cannot be set in …/.cfnlintrc`), so neither
belongs there. `~/.cfnlintrc` may still set `append_rules`.

To run the rules locally, use
`cfn-lint --append-rules cfn_lint_ax.rules cfn_lint_ax.serverless_rules`, or put `append_rules` in
`~/.cfnlintrc`, which also covers the editor plugin.

The rules are listed in [aexeagmbh/cfn-lint-rules](https://github.com/aexeagmbh/cfn-lint-rules).


## Example usage

```yaml
uses: aexeagmbh/gh-actions-cfn-lint@main
with:
  working-directory: ./foo
```
