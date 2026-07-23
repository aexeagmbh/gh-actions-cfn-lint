# gh-actions-cfn-lint

This action runs cfn-lint. The additional rules from cfn-lint-ax are available.


## Inputs

### `working-directory`

**Optional** Specifies the working directory where the commands are run.


## Example usage

```yaml
uses: aexeagmbh/gh-actions-cfn-lint@main
with:
  working-directory: ./foo
```
