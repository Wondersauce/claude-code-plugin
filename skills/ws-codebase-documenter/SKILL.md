---
name: ws-codebase-documenter
description: Generate and maintain comprehensive codebase documentation optimized for AI consumption. Optionally syncs to a Docusaurus site. Use when asked to document a codebase, generate API documentation, create docs for AI agents, maintain documentation after code changes, or sync docs to Docusaurus. Supports Node.js/TypeScript, Python, Go, Rust, .NET, Java, and PHP projects.
argument-hint: "[bootstrap|update]"
---

# Codebase Documenter

Generate structured documentation designed for consumption by AI assistants and code agents working on dependent projects.

## Workflow

### 1. Determine Mode

Check if `documentation/config.json` exists:
- **Does not exist**: Bootstrap Mode (first run)
- **Exists**: Incremental Mode (subsequent runs)

### 2. Bootstrap Mode

Execute when `documentation/config.json` is absent.

#### 2.1 Detect Stack

Check for these files in order:

| File | Stack |
|------|-------|
| `package.json` | nodejs |
| `requirements.txt`, `pyproject.toml`, `setup.py` | python |
| `go.mod` | go |
| `Cargo.toml` | rust |
| `*.csproj`, `*.sln` | dotnet |
| `pom.xml`, `build.gradle` | java |
| `composer.json` | php |

#### 2.2 Create Config

Write `documentation/config.json`:

```json
{
  "stack": "[detected-stack]",
  "exclude": [
    "**/*.test.*",
    "**/*.spec.*",
    "**/test/**",
    "**/tests/**",
    "**/__tests__/**",
    "**/node_modules/**",
    "**/vendor/**",
    "**/dist/**",
    "**/build/**",
    "**/target/**"
  ],
  "include_inline_examples": true,
  "include_architecture_diagrams": true,
  "docusaurus": null
}
```

If Docusaurus sync is requested, set the `docusaurus` field:

```json
{
  "docusaurus": {
    "repo": "git@github.com:org/docs-site.git",
    "branch": "main",
    "docs_path": "docs/api",
    "sidebar_label": "API Reference"
  }
}
```

#### 2.3 Read Stack Reference

Load the appropriate reference file:
- `references/stacks/[stack].md`

This provides patterns for identifying public/private API, documentation comments, error handling, and project structure conventions.

#### 2.4 Scan Codebase

1. List all source files (respecting `exclude` patterns)
2. Identify public API surface using stack-specific patterns
3. Categorize items: functions, types, errors, features
4. Extract documentation comments
5. Map relationships between items

#### 2.5 Generate Documentation

Read `references/doc-templates.md` for exact formats.

Create directory structure:
```
documentation/
├── config.json
├── .docstate
├── overview.md
├── architecture.md
├── public/
│   ├── _index.md
│   ├── features/
│   ├── api/
│   ├── functions/
│   ├── types/
│   └── errors/
└── private/
    ├── _index.md
    ├── functions/
    ├── types/
    └── utils/
```

Generate files:
1. `overview.md` - Project purpose, entry points, quick start
2. `architecture.md` - Component diagrams, data flow, design patterns
3. Function docs - One file per function in appropriate directory
4. Type docs - One file per type
5. Error docs - Error hierarchy and patterns
6. Index files - `_index.md` in each directory

#### 2.6 Update Claude Code Instructions

Create or update `CLAUDE.md` in the project root to reference the documentation. If the file exists, append to it; otherwise create it.

Add this section:

```markdown
## Codebase Documentation

This project has AI-optimized documentation in the `documentation/` folder.

Before making changes to this codebase:
1. Read `documentation/overview.md` for project purpose and entry points
2. Read `documentation/architecture.md` for system design and data flow
3. Check `documentation/public/_index.md` for the public API surface

When modifying existing code:
- Check the relevant function/type doc in `documentation/public/` or `documentation/private/`
- Note any error handling patterns in `documentation/public/errors/`

When adding new public APIs:
- Follow patterns documented in `documentation/architecture.md`
- Ensure consistency with existing APIs in `documentation/public/`
```

#### 2.7 Write State

Create `documentation/.docstate`:
```json
{
  "last_commit": "[current HEAD SHA]",
  "last_run": "[ISO timestamp]"
}
```

Get current HEAD:
```bash
git rev-parse HEAD
```

### 3. Incremental Mode

Execute when `documentation/config.json` exists.

#### 3.1 Load State

Read `documentation/config.json` and `documentation/.docstate`.

#### 3.2 Get Changes

```bash
git diff --name-status [last_commit]..HEAD
```

Filter to source files only (exclude test files, config, etc).

#### 3.3 Analyze Changes

For each changed file:

1. Get the diff:
   ```bash
   git diff [last_commit]..HEAD -- [file]
   ```

2. Identify semantic changes:
   - New functions/types/classes added
   - Signatures modified (parameters, return types)
   - Items removed or deprecated
   - Error handling changes
   - Documentation comment updates

#### 3.4 Update Documentation

For each semantic change:

| Change Type | Action |
|-------------|--------|
| New item | Create new doc file |
| Modified signature | Update existing doc |
| Removed item | Mark deprecated or delete |
| New error | Add to error docs |

Also update:
- Relevant `_index.md` files
- `architecture.md` if structure changed
- Cross-references in related docs

#### 3.5 Verify Claude Code Instructions

Check that `CLAUDE.md` contains the documentation reference section. If missing (e.g., file was reset or recreated), add it using the template from Bootstrap Mode step 2.6.

#### 3.6 Update State

Update `documentation/.docstate` with new HEAD and timestamp.

### 4. Create PR

#### 4.1 Branch Naming

Format: `docs/auto-update-YYYY-MM-DD-HH:MM`

If branch exists, append `-2`, `-3`, etc.

```bash
BRANCH="docs/auto-update-$(date +%Y-%m-%d-%H:%M)"
git checkout -b "$BRANCH"
```

#### 4.2 Commit Changes

```bash
git add documentation/
git commit -m "docs: update documentation for [commit range]"
```

#### 4.3 Push and Create PR

```bash
git push -u origin "$BRANCH"
```

Create PR with description summarizing changes (see PR Description template in `references/doc-templates.md`).

### 5. Docusaurus Sync (Optional)

Execute only if `config.docusaurus` is configured. See `references/docusaurus.md` for detailed patterns.

#### 5.1 Clone/Update Docusaurus Repo

```bash
DOCS_REPO="[config.docusaurus.repo]"
DOCS_BRANCH="[config.docusaurus.branch]"
DOCS_PATH="[config.docusaurus.docs_path]"

# Clone to temp directory if not exists
if [ ! -d ".docusaurus-sync" ]; then
  git clone --depth 1 -b "$DOCS_BRANCH" "$DOCS_REPO" .docusaurus-sync
else
  cd .docusaurus-sync && git pull && cd ..
fi
```

#### 5.2 Transform Documentation

For each markdown file in `documentation/`:

1. Add Docusaurus frontmatter:
   ```yaml
   ---
   id: [filename-without-extension]
   title: [extracted-from-h1]
   sidebar_label: [short-title]
   sidebar_position: [order]
   ---
   ```

2. Convert internal links from `../types/Foo.md` to Docusaurus paths

3. Copy to `.docusaurus-sync/[docs_path]/`

#### 5.3 Generate Sidebar

Create or update `.docusaurus-sync/[docs_path]/_category_.json` for each directory:

```json
{
  "label": "[directory-name-titlecased]",
  "position": [order],
  "collapsed": false
}
```

Generate `sidebars.js` entry if needed (see `references/docusaurus.md`).

#### 5.4 Commit and Push to Docusaurus Repo

```bash
cd .docusaurus-sync
git add .
git commit -m "docs: sync from [source-repo] @ [commit-sha]"
git push origin "$DOCS_BRANCH"
```

Or create a PR in the Docusaurus repo if preferred.

#### 5.5 Cleanup

```bash
rm -rf .docusaurus-sync
```

## Documentation Standards

### AI Optimization

Documentation must be optimized for AI consumption:

1. **Structured data over prose** - Use tables for parameters, errors, options
2. **Explicit types** - Always include full type signatures
3. **Complete examples** - Show imports, setup, and usage
4. **Error recovery** - Document how to handle each error
5. **Cross-references** - Link to related functions and types
6. **No ambiguity** - Specify defaults, constraints, edge cases

### Public vs Private

**Public** (`documentation/public/`):
- Exported functions, classes, types
- Items intended for external use
- API surface for dependent projects

**Private** (`documentation/private/`):
- Internal implementation details
- Helper functions
- Useful for understanding internals but not for direct use

### When to Document Private Items

Document private items when:
- They're complex and non-obvious
- They're frequently modified
- Understanding them helps debug issues
- They contain important business logic

Skip documenting:
- Trivial helpers (less than 5 lines)
- Generated code
- Boilerplate

## Error Handling

If documentation generation fails:
1. Do not commit partial changes
2. Log the error
3. Exit with non-zero status

Common issues:
- Cannot detect stack: Ask user to set `stack` in config.json
- Cannot parse source files: Skip unparseable files, continue with others
- Git operations fail: Ensure clean working directory

## Reference Files

Load these as needed:

| File | When to Load |
|------|--------------|
| `references/stacks/[stack].md` | After detecting or reading stack |
| `references/doc-templates.md` | When generating any documentation |
| `references/docusaurus.md` | When `config.docusaurus` is configured |
