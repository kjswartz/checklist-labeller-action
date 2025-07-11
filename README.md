# Checklist Labeller Action

This action automatically swaps GitHub issue labels when all checkboxes in a specified checklist section are completed.

## Usage

```yaml
name: Checklist Approval
on:
  issue_comment:
    types: [created, edited]

jobs:
  checklist-approval:
    runs-on: ubuntu-latest
    steps:
      - uses: kjswartz/checklist-labeller-action@main
        id: checklist-check
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          issue-number: ${{ github.event.issue.number }}
          text-body: ${{ github.event.comment.body }}
          issue-labels: ${{ toJson(github.event.issue.labels) }}
          run-check-label: 'P-1'
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
| `issue-labels` | JSON array of issue labels | Yes | - |
| `run-check-label` | Label indicating checklist present | Yes | - |
| `incomplete-label` | Label indicating checklist incomplete | No | - |
| `completed-label` | Label to add when checklist is completed | No | - |
| `require-checkbox-pattern` | Regex pattern for checkbox matching | No | `^\- \[ \]` |
| `completed-checkbox-pattern` | Label to add when checklist is completed | No | `^\- \[[xX]\]` |

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
