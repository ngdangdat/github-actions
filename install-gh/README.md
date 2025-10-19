# Install GitHub CLI Action

A reusable GitHub Action to install `gh` (GitHub CLI) on Linux, macOS, and Windows runners.

## Features

- Supports Linux, macOS, and Windows runners
- Checks if `gh` is already installed to avoid unnecessary reinstalls
- Verifies installation by displaying the version

## Usage

To use this action in your workflow, reference it in your workflow file:

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Install GitHub CLI
    uses: <your-username>/github-actions@main

  - name: Use gh
    run: gh --version
```

### Example Workflow

```yaml
name: Example Workflow

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install GitHub CLI
        uses: <your-username>/github-actions@main

      - name: Authenticate with GitHub
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          echo "$GITHUB_TOKEN" | gh auth login --with-token

      - name: Create an issue
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          gh issue create --title "Test Issue" --body "Created by GitHub Actions"
```

### Multi-platform Example

```yaml
name: Multi-platform Test

on: [push]

jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4

      - name: Install GitHub CLI
        uses: <your-username>/github-actions@main

      - name: Verify installation
        run: gh --version
```

## How It Works

- **Linux**: Installs from the official GitHub CLI apt repository
- **macOS**: Uses Homebrew to install `gh`
- **Windows**: Uses Chocolatey to install `gh`

## Notes

- The action checks if `gh` is already installed before attempting installation
- On GitHub-hosted runners, `gh` is often pre-installed, so the action will detect this and skip installation
- Requires sudo/admin privileges on the runner (standard for GitHub-hosted runners)

## License

This action is available for use under your preferred license.
