# Contributing

Thanks for your interest in improving this PR review skill.

## How to Contribute

1. Fork the repo
2. Create a branch: `feat/your-feature` or `fix/your-fix`
3. Make your changes
4. Test the skill by running `/review` on a real PR
5. Open a PR with a clear description of what you changed and why

## Adding New Rules

New review rules go in `commands/review.md`. Follow the existing pattern:

- **Backend rules**: `B1`, `B2`, ... `B21` — add as `B22`, `B23`, etc.
- **Frontend rules**: `F1`, `F2`, ... `F15` — add as `F16`, `F17`, etc.
- **Cross-cutting rules**: `X1`, `X2`, ... `X4` — add as `X5`, `X6`, etc.

Each rule should have:
- A clear, descriptive title
- Checkboxes for each specific check
- Explanation of WHY the rule matters

## Updating Severity Classification

If you add a new rule, update the severity classification in Phase 4 to include it.

## Testing

Run the skill against real PRs to verify:
- Rules trigger correctly on violations
- No false positives on clean code
- Comments are clear and actionable

## Code of Conduct

Be respectful, constructive, and helpful. Same standards we enforce in PR reviews.
