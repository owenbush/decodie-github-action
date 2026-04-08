<p align="center"><img src="assets/decodie-logo.png" alt="Decodie" width="200"></p>

# Decodie GitHub Action

Automatically analyze changed files in pull requests and generate [Decodie](https://decodie.owenbush.dev) learning entries using the same skill that powers Claude Code and the VSCode extension.

Built on top of [claude-code-action](https://github.com/anthropics/claude-code-action) — runs the official [Decodie skill](https://github.com/owenbush/decodie-skill) (`/decodie:analyze`) in your CI pipeline.

## Quick Start

```yaml
name: Decodie
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.head_ref }}
      - uses: owenbush/decodie-github-action@v1
        with:
          # Use ONE of:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          # claude-code-oauth-token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

## What It Does

1. **Installs the Decodie skill** — the same `/decodie:analyze` command used in Claude Code
2. **Runs Claude Code** via [claude-code-action](https://github.com/anthropics/claude-code-action) to analyze changed files
3. **Generates structured entries** — patterns, decisions, rationale, code snippets, key concepts
4. **Commits entries to `.decodie/`** in the PR branch for use with the VSCode extension and web UI
5. **Posts a PR comment** summarizing the analysis with collapsible sections per file

Because it uses the real Decodie skill, entries are identical in format and quality to what you get from `/decodie:analyze` in Claude Code or "Analyze File" in the VSCode extension.

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `anthropic-api-key` | Anthropic API key (provide this or `claude-code-oauth-token`) | `""` |
| `claude-code-oauth-token` | Claude Code OAuth token (provide this or `anthropic-api-key`) | `""` |
| `mode` | Analysis mode: `selective` or `exhaustive` | `selective` |
| `model` | Claude model to use | `claude-sonnet-4-6` |
| `max-files` | Maximum number of changed files to analyze (0 for no limit) | `10` |
| `include` | Comma-separated glob patterns for files to include | `""` |
| `exclude` | Comma-separated glob patterns for files to exclude | `""` |
| `comment` | Whether to post a PR comment with the analysis summary | `true` |
| `commit` | Whether to commit generated entries to `.decodie/` in the PR branch | `true` |
| `github-token` | GitHub token with repo and pull request permissions | `${{ github.token }}` |

### Authentication

Provide **one** of:
- **`anthropic-api-key`** — pay-per-token from [console.anthropic.com](https://console.anthropic.com/)
- **`claude-code-oauth-token`** — uses your Claude Pro/Max subscription (run `claude setup-token` to generate one)

Store whichever you use as a GitHub repository secret.

## Usage Examples

### Basic

```yaml
- uses: actions/checkout@v4
  with:
    ref: ${{ github.head_ref }}
- uses: owenbush/decodie-github-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
```

### With File Filtering

```yaml
- uses: owenbush/decodie-github-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    include: "src/**/*.ts,lib/**/*.ts"
    exclude: "**/*.test.ts,**/*.spec.ts"
    max-files: 5
```

### Comment Only (no commit)

```yaml
- uses: owenbush/decodie-github-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    commit: 'false'
```

### Commit Only (no comment)

```yaml
- uses: owenbush/decodie-github-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    comment: 'false'
```

### Exhaustive Mode

```yaml
- uses: owenbush/decodie-github-action@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    mode: exhaustive
    max-files: 3
```

## Configuration

### File Filtering

By default, the Decodie skill skips binaries, lock files, vendor directories, and other non-source files. Use `include` and `exclude` to further control which files are analyzed. Both accept comma-separated glob patterns.

### Cost Control

Each analyzed file makes Claude API calls. To keep costs predictable:

- Use `max-files` to cap the number of files per PR (default: 10)
- Use `include` to limit analysis to specific directories
- Use `selective` mode (the default) which generates 3-5 entries per file
- `exhaustive` mode documents every meaningful pattern and costs more

### Permissions

The workflow needs these permissions:

```yaml
permissions:
  contents: write       # To commit .decodie/ entries
  pull-requests: write  # To post PR comments
```

Use `actions/checkout` with `ref: ${{ github.head_ref }}` so commits go to the PR branch.

## How It Works

This action is a composite wrapper around [anthropics/claude-code-action](https://github.com/anthropics/claude-code-action). It:

1. Installs the Decodie skill via `npx @owenbush/decodie-ui install-skill`
2. Runs `claude-code-action` with a prompt that instructs Claude to use `/decodie:analyze` on the PR's changed files
3. The skill handles everything: file discovery, analysis, entry generation, schema compliance, duplicate detection, and writing to `.decodie/`

This means the entries are generated by the exact same code that runs in Claude Code and the VSCode extension — same prompts, same schema, same quality.

## The Decodie Ecosystem

| Project | Description |
|---------|-------------|
| [decodie-skill](https://github.com/owenbush/decodie-skill) | Claude Code skill for real-time and retroactive code documentation |
| [decodie-ui](https://github.com/owenbush/decodie-ui) | Web-based browser with lessons, progress tracking, and Q&A |
| [decodie-vscode](https://marketplace.visualstudio.com/items?itemName=owenbush.decodie-vscode) | VSCode extension for browsing entries in your editor |
| [decodie-ddev](https://github.com/owenbush/decodie-ddev) | DDEV add-on for local development |
| [decodie-core](https://github.com/owenbush/decodie-core) | Shared data layer (types, parser, reference resolver) |
| [decodie.owenbush.dev](https://decodie.owenbush.dev) | Project homepage |

## License

MIT
