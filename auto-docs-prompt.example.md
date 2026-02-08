# Auto-Docs Update Prompt

You are updating engineering documentation based on recent changes to a source repository.

## Context

- **Documentation file:** `$DOC_FILE`
- **Source repository:** `$REPOSITORY` (checked out at `$SOURCE_PATH`)
- **Scope:** $SCOPE
- **Changed paths:** `$CHANGED_PATHS`
- **Commit SHA:** `$SHA`

## Instructions

1. Read the current documentation file at `$DOC_FILE`
2. Explore the source repository at `$SOURCE_PATH`, focusing on the areas indicated by the changed paths
3. Compare the current documentation against the actual code to identify discrepancies
4. Update **only** the sections that are affected by the changes — do not rewrite sections that are still accurate
5. If you find no meaningful documentation changes are needed (e.g., the code change was internal and doesn't affect documented conventions), output `NO_CHANGES_NEEDED` and stop

## Rules

- **Preserve the existing document structure** — same heading hierarchy, same section order
- **Use Avoid/Prefer code block pairs** when documenting conventions (matching the pattern used across all standards docs)
- **Include file path references** when citing examples from the source repo (e.g., "Example from `src/api/handler.go`")
- **Do not remove content** that is still accurate in the source repo — only update what has actually changed
- **Do not add speculative content** — only document patterns that exist in the code
- **Keep the same tone and depth** as the existing document
- **British English** spelling is recommended (colour, organised, capitalise, behaviour) — remove or change this rule to match your team's convention
- Replace the line below with your linting command, or remove it if you don't lint docs:
- `# Replace with your linting command, e.g.: npx markdownlint-cli2 "$DOC_FILE"`

## Output

Do not commit your changes — leave all modifications as uncommitted changes in the working tree. The CI pipeline will handle committing.
