# Setup Git as GitHub App

A GitHub Action that configures Git to authenticate as a GitHub App, allowing your workflows to perform Git operations with elevated permissions and better security than using personal access tokens.

## Features

- Generate GitHub App installation access tokens automatically
- Configure Git with proper credentials for authenticated operations
- Auto-detect installation ID or use a provided one
- Customize Git user name and email
- Output token for use in subsequent steps
- Secure token handling with automatic masking in logs

## Why Use GitHub Apps?

- **Better security**: Tokens are scoped to specific permissions and repositories
- **Higher rate limits**: GitHub Apps get higher API rate limits
- **Fine-grained permissions**: Only grant the permissions your workflow needs
- **Audit trail**: Actions appear as the app, not a personal account
- **No personal token needed**: Reduces security risks

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
   - Installation ID (optional, can be auto-detected)

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
        uses: your-username/setup-git-as-github-app@v1
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

### Advanced Example

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
        uses: your-username/setup-git-as-github-app@v1
        with:
          app-id: ${{ secrets.APP_ID }}
          private-key: ${{ secrets.APP_PRIVATE_KEY }}
          installation-id: ${{ secrets.INSTALLATION_ID }}
          git-user-name: 'My Bot'
          git-user-email: 'bot@example.com'

      - name: Use the generated token
        run: |
          echo "Token generated for installation: ${{ steps.git-setup.outputs.installation-id }}"
          # Use token for API calls
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
| `installation-id` | GitHub App installation ID | No | Auto-detected |
| `git-user-name` | Git user.name to configure | No | `github-actions[bot]` |
| `git-user-email` | Git user.email to configure | No | `github-actions[bot]@users.noreply.github.com` |

## Outputs

| Output | Description |
|--------|-------------|
| `token` | Generated GitHub App installation access token |
| `installation-id` | GitHub App installation ID used |

## How to Store Secrets

1. Go to your repository Settings → Secrets and variables → Actions
2. Add the following secrets:
   - `APP_ID`: Your GitHub App ID
   - `APP_PRIVATE_KEY`: Your GitHub App private key (entire PEM file content)
   - `INSTALLATION_ID` (optional): Your installation ID

## Permissions Required

Your GitHub App needs the following permissions depending on your use case:

- **Contents**: Read and write (for Git operations)
- **Metadata**: Read (automatically granted)
- **Pull requests**: Read and write (if creating/updating PRs)

## Troubleshooting

### Authentication fails
- Verify your App ID is correct
- Ensure the private key is the complete PEM file content
- Check that the app is installed on the repository

### Installation ID not found
- Verify the app is installed on the repository or organization
- Provide the installation ID explicitly if auto-detection fails

### Git push fails
- Ensure the App has Contents: Write permission
- Verify the repository exists and is accessible

## Security Considerations

- Never commit private keys to your repository
- Always use GitHub Secrets to store sensitive information
- The generated token is automatically masked in GitHub Actions logs
- Tokens expire after 1 hour (GitHub App tokens are short-lived)

## License

MIT License - see [LICENSE](LICENSE) file for details

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Support

If you encounter any issues or have questions, please [open an issue](../../issues).
