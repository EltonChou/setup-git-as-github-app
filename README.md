# Setup Git as GitHub App

A GitHub Action that configures Git to authenticate as a GitHub App.

## Features

- Generate GitHub App installation access tokens automatically
- Automatically set Git user name and email based on your GitHub App
- Output token, user name, and email for use in subsequent steps

## Prerequisites

1. Create a GitHub App:
   - Go to Settings → Developer settings → GitHub Apps → New GitHub App
   - Give it a name and description
   - Set required permissions (e.g., Contents: Read & Write for Git operations)
   - Generate a private key (download and save securely)

2. Install the app on your repository or organization

3. Note down:
   - App ID (found in app settings)
   - Private key (from downloaded .pem file)

## Usage

### Basic Example

```yaml
name: Example Workflow

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Git as GitHub App
        uses: EltonChou/setup-git-as-github-app@main
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Make changes and commit
        run: |
          echo "New content" > file.txt
          git add file.txt
          git commit -m "Update file"
          git push
```

### Advanced Example with Outputs

```yaml
name: Advanced Example

on:
  workflow_dispatch:

jobs:
  update-repo:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Git as GitHub App
        id: git-setup
        uses: EltonChou/setup-git-as-github-app@main
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}

      - name: Display configured Git user
        run: |
          echo "Git user name: ${{ steps.git-setup.outputs.user-name }}"
          echo "Git user email: ${{ steps.git-setup.outputs.user-email }}"

      - name: Use the generated token for API calls
        run: |
          curl -H "Authorization: Bearer ${{ steps.git-setup.outputs.token }}" \
               https://api.github.com/repos/${{ github.repository }}

      - name: Perform Git operations
        run: |
          git checkout -b feature-branch
          echo "Changes" >> README.md
          git add README.md
          git commit -m "Automated update"
          git push origin feature-branch
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `app-id` | GitHub App ID | Yes | - |
| `private-key` | GitHub App private key (PEM format) | Yes | - |

## Outputs

| Output | Description |
|--------|-------------|
| `token` | Generated GitHub App installation access token |
| `user-name` | Configured Git user name (format: `app-slug[bot]`) |
| `user-email` | Configured Git user email (format: `user-id+app-slug[bot]@users.noreply.github.com`) |

## How It Works

1. The action uses `actions/create-github-app-token@v2` to generate an installation access token for your GitHub App
2. It retrieves the GitHub App's user ID using the GitHub API
3. It automatically configures Git with:
   - User name: `<app-slug>[bot]` (e.g., `my-app[bot]`)
   - User email: `<user-id>+<app-slug>[bot]@users.noreply.github.com`
4. Git is configured locally for the current repository

## How to Store Secrets

1. Go to your repository Settings → Secrets and variables → Actions
2. Add the following secrets:
   - `APP_ID`: Your GitHub App ID
   - `APP_PRIVATE_KEY`: Your GitHub App private key (entire PEM file content)

## Permissions Required

Your GitHub App needs the following permissions depending on your use case:

- **Contents**: Read and write (for Git operations)
- **Metadata**: Read (automatically granted)
- **Pull requests**: Read and write (if creating/updating PRs)

## License

MIT License - see [LICENSE](LICENSE) file for details

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

If you encounter any issues or have questions, please [open an issue](../../issues).
