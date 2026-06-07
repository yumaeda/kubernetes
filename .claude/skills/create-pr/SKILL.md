---
name: create-pr
description: Create a new Pull Request on GitHub from local changes
metadata:
  type: skill
---

# Create Pull Request Skill

Use this skill to create a new Pull Request on GitHub from local changes.

## Steps

1. Change to the repository root directory
2. Stage all changes: `git add .`
3. Commit changes on the current branch: `git commit -m "<PR_COMMENT>"`
4. Create a new branch from the commit and switch to it: `git checkout -b <BRANCH_NAME>`
5. Push and create upstream tracking: `git push origin -u HEAD`
6. Open a PR via `gh pr create`

## Branch Naming

Generate a branch name based on the local changes:
- Prefix with type: `feat/`, `fix/`, `chore/`, `refactor/`, `docs/`
- Follow kebab-case
- Example: `feat/add-auth-flow`, `fix/header-alignment`

## Commit Message

Generate a concise commit message based on the local changes:
- Imperative mood ("Add feature", not "Added feature")
- Max 72 characters
- Focus on the "why" not the "what"

## PR Title

Follow conventional commit format:
- `feat: <description>`
- `fix: <description>`
- `chore: <description>`
- `refactor: <description>`
- `docs: <description>`

## PR Body

Generate a PR body with this structure:

```
## Summary
<1-3 bullet points describing the changes>

## Change Type
<feat / fix / refactor / chore / docs>

## Files Changed
- List of key files modified

## Testing
<How to verify the changes>
```

## Pre-flight Checks

Before creating the PR:
- Ensure working tree is clean (no uncommitted changes besides staged ones)
- Verify the remote exists (`git remote -v`)
- Check if base branch is up to date with `main` — if not, suggest rebasing first

## GitHub PR Creation

Use `gh pr create` with:
- `--base main` (unless project uses a different default)
- `--head <BRANCH_NAME>`
- `--title "<PR_TITLE>"`
- Pass body via heredoc to preserve formatting

Example:
```bash
gh pr create \
  --base main \
  --head <BRANCH_NAME> \
  --title "<PR_TITLE>" \
  --body "$(cat <<'EOF'
## Summary
- ...

## Change Type
<type>

## Files Changed
- ...

## Testing
- ...
EOF
)"
```

## Error Handling

- If `gh` is not authenticated, guide user through `gh auth login`
- If branch already exists, suggest `git checkout <BRANCH_NAME>` and continuing
- If remote push fails, check for unmerged changes or stale branches
