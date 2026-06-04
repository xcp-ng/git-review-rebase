# git-review-rebase

An interactive TUI (Terminal User Interface) tool for reviewing rebased git branches side-by-side.

![git-review-rebase demo](imgs/git-review-rebase-demo.gif)

## Features

- Side-by-side diff viewing for rebased commits
- Interactive commit matching between branches
- Syntax highlighting with token-level diff highlighting
- Git blame integration for tracking commit origins
- Fuzzy search across commits
- Flexible filtering by commit match types
- Possibility to add notes to dropped commits

### Commit match icons

  | Icon | Meaning                                   |
  | ---- | ----------------------------------------- |
  | `=`  | Same commit (identical SHA1               |
  | `~`  | Loose match (same title, patchid changed) |
  | `⤶`  | Already present in new upstream           |
  | `✗`  | Dropped during rebase                     |
  | `✚`  | Added in rebase                           |

> **Patchid** is a checksum of a commit's diff (`git patch-id`). Two commits with different
> SHA1 (e.g. after rebase) that introduce the same code changes will have the same patchid.
> This allows the tool to match commits across a rebase even when their SHA-1s change.

## Installation

### From source

```bash
pip install -e .
```

## Usage

```bash
git-review-rebase <base>..<left-branch> <onto_base>..<right-branch>
```

### Options

- `--repository PATH`: Path to git repository (default: current directory)
- `--no-cache`: Disable patchid caching

## Development

### Setup

```bash
# Install in development mode with dev dependencies
pip install -e ".[dev]"
```

### Code formatting and linting

```bash
black src/
ruff check src/
flake8 src/
```

### Type checking

```bash
mypy src/
pyright ./
```

## License

GPL-v2

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
