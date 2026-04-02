# aloop-spine - Claude Code Instructions

## Repo Purpose
This is the operational architecture spine for ALoop. It contains numbered markdown files (00-10) that AI coding agents read for context when editing any ALoop repository.

## Editing Rules
- Each numbered file must have a YAML-style header: title, last-updated, primary-audience, source
- Keep files factual and current - no aspirational content, no "we plan to" language
- When updating a file, update the last-updated date in the header
- Do not delete audit/ artifacts - they are provenance records

## File Format
Every numbered file (00-10) follows this header format:

    ---
    title: [File Title]
    last-updated: YYYY-MM-DD
    primary-audience: all | frontend | backend | operations
    source: consensus (audit/outside-in + audit/inside-out)
    ---

## Commit Convention
- Commit messages: `[spine] Brief description`
- Stage specific files, never git add -A

## No Em Dashes
Never use em dashes or double dashes in written content. Use single hyphens, commas, periods, semicolons, or line breaks instead.
