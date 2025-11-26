# create-pull-request-poc

A Proof of Concept repository demonstrating how to use GitHub Actions to create pull requests with signed commits using a GitHub App.

## Overview

This PoC verifies that commits can be signed using a GitHub App with two different GitHub Actions:

1. **[peter-evans/create-pull-request@v7](https://github.com/peter-evans/create-pull-request)** - Creates pull requests with signed commits using `sign-commits: true`
2. **[googleapis/release-please-action@v4](https://github.com/googleapis/release-please-action)** - Automates releases with Conventional Commit Messages using signed commits

When using a GitHub App token, commits are signed as `<application-name>[bot]` and appear as "Verified" on GitHub.

## Setup

### 1. Create a GitHub App

1. Go to **Settings > Developer settings > GitHub Apps > New GitHub App**
2. Configure the app with:
   - **GitHub App name**: Choose a name (e.g., `your-org-signed-commits`)
   - **Homepage URL**: Your repository URL
   - **Webhook**: Uncheck "Active" (not needed)
   - **Repository permissions**:
     - **Contents**: Read & write
     - **Pull requests**: Read & write
     - **Workflows**: Read & write (only if PRs may contain workflow changes)
3. Click "Create GitHub App"
4. Note the **App ID** displayed on the app's settings page
5. Generate a **Private key** and download it

### 2. Install the GitHub App

1. From the GitHub App settings, click "Install App"
2. Select the repository (or organization) where you want to use it
3. Grant access to this repository

### 3. Configure Repository Secrets

Add the following secrets to your repository (**Settings > Secrets and variables > Actions**):

| Secret Name | Description |
|-------------|-------------|
| `APP_ID` | The App ID from your GitHub App settings |
| `APP_PRIVATE_KEY` | The contents of the private key file (`.pem`) you downloaded |

### 4. Enable Workflow Permissions

Ensure that GitHub Actions has permission to create pull requests:
1. Go to **Settings > Actions > General**
2. Under "Workflow permissions", enable "Allow GitHub Actions to create and approve pull requests"

## Usage

### Workflow 1: Manual Trigger (Create PR with Signed Commits)

1. Go to **Actions > Create PR with Signed Commits**
2. Click "Run workflow"
3. Optionally enter a custom message
4. Click "Run workflow"

The workflow will:
1. Generate a token from your GitHub App
2. Make a change to `updates.txt`
3. Create a pull request with signed commits
4. The commits will show as "Verified" because they are signed by the GitHub App

### Workflow 2: Release Please (Automatic on Push to Main)

This workflow runs automatically on every push to the `main` branch:

1. Release Please analyzes commits using [Conventional Commits](https://www.conventionalcommits.org/) format
2. When releasable changes are detected, it creates/updates a Release PR
3. When the Release PR is merged, it creates a GitHub Release with the appropriate version tag
4. All commits are signed using the GitHub App token

## Verification

After running the workflow, check the created pull request:
1. Open the pull request
2. Click on the commit(s)
3. You should see a "Verified" badge next to each commit
4. Clicking on "Verified" will show that the commit was signed by your GitHub App

## Workflow Output

The workflow outputs the following information:
- `pull-request-number`: The PR number
- `pull-request-url`: Direct link to the PR
- `pull-request-commits-verified`: Whether GitHub considers the commits verified (`true`/`false`)

## Key Configuration

### peter-evans/create-pull-request

The essential configuration in the workflow for signed commits:

```yaml
- uses: actions/create-github-app-token@v2
  id: generate-token
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}

- uses: peter-evans/create-pull-request@v7
  with:
    token: ${{ steps.generate-token.outputs.token }}
    sign-commits: true
```

### release-please-action

The essential configuration for Release Please with signed commits:

```yaml
- uses: actions/create-github-app-token@v2
  id: generate-token
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}

- uses: googleapis/release-please-action@v4
  with:
    token: ${{ steps.generate-token.outputs.token }}
    release-type: simple
```

When using a GitHub App token instead of the default `GITHUB_TOKEN`, Release Please will create commits that are signed by the GitHub App.

## Notes

- When `sign-commits: true` is used, the `committer` and `author` inputs are ignored
- Commit signing only works with bot-generated tokens (GITHUB_TOKEN or GitHub App tokens), not with Personal Access Tokens (PATs)
- The GitHub API has a 40MiB limit for git blobs when signing commits

## References

- [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request)
- [googleapis/release-please-action](https://github.com/googleapis/release-please-action)
- [release-please](https://github.com/googleapis/release-please)
- [Commit signature verification for bots](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#commit-signature-verification-for-bots)
- [Authenticating with GitHub App tokens](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#authenticating-with-github-app-generated-tokens)
- [actions/create-github-app-token](https://github.com/actions/create-github-app-token)
- [Conventional Commits](https://www.conventionalcommits.org/)