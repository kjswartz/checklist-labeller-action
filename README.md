# Checklist Labeller Action

This action automatically swaps GitHub issue labels when all checkboxes in a specified checklist section are completed. If the checklist was previously completed and has the completed-label attached and is then unchecked, the completed-label will be removed and the incomplete-label will be added back to the issue. Once the checklist is completed again, the incomplete-label will be removed and the completed-label will be added back. You do not need to include the completed-label, or incompleted-label. If you only include one of them, then just that label will be added/removed based on the completion status. The `run-check-label` is an optional variable that if supplied, will only run the checklist completion search and label updates, if the label is found on the issue. 

## Usage

```yaml
name: Checklist Completion Labeller
on:
  issues:
    types: [edited]

jobs:
  checklist-completion:
    runs-on: ubuntu-latest
    steps:
      - uses: kjswartz/checklist-labeller-action@main
        id: checklist-check
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          issue-number: ${{ github.event.issue.number }}
          text-body: ${{ github.event.issue.body }}
          issue-labels: ${{ toJson(github.event.issue.labels) }}
          incomplete-label: 'verification-needed'
          completed-label: 'verified'
          checklist-key: 'p1-checklist'

      - name: Report Results
        run: |
          SUMMARY_MSG="### Status\n\n${{ steps.checklist-check.outputs.status }}\n\n### Message\n\n${{ steps.checklist-check.outputs.message }}"
          echo -e "$SUMMARY_MSG"
          echo -e "$SUMMARY_MSG" >> "$GITHUB_STEP_SUMMARY"
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token for API access | Yes | `${{ github.token }}` |
| `issue-number` | Issue number to process | Yes | - |
| `text-body` | Text body to check | Yes | - |
| `checklist-key` | Key to identify checklist section | Yes | - |
| `issue-labels` | JSON array of issue labels | No | - |
| `run-check-label` | Label indicating checklist present. Provides early exit to the script if label is not found. | No | - |
| `incomplete-label` | Label indicating checklist incomplete | No | - |
| `completed-label` | Label to add when checklist is completed | No | - |
| `require-checkbox-pattern` | Regex pattern for checkbox matching | No | `^\- \[ \]` |
| `completed-checkbox-pattern` | Regex pattern for completed checkbox matching | No | `^\- \[[xX]\]` |

## Outputs

| Output | Description |
|--------|-------------|
| `status` | Status: `completed`, `incomplete`, `no-checklist`, `no-checkboxes`, `no-check-label`, `skipped` |
| `message` | Descriptive message about the result |

## Checklist Format

The action looks for checklists between HTML comments:

```markdown
<!-- key="p1-checklist" value="start" -->
- [x] Item 1 completed
- [ ] Item 2 not completed
- [x] Item 3 completed
<!-- key="p1-checklist" value="end" -->
```
