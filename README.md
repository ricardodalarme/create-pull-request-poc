# create-pull-request-poc

A Proof of Concept repository demonstrating how to use [peter-evans/create-pull-request@v7](https://github.com/peter-evans/create-pull-request) GitHub Action to create pull requests with signed commits using a GitHub App.

## Overview

This PoC verifies that commits can be signed using a GitHub App with the `peter-evans/create-pull-request@v7` action. When using a GitHub App token with `sign-commits: true`, commits are signed as `<application-name>[bot]` and appear as "Verified" on GitHub.

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

### Manual Trigger

1. Go to **Actions > Create PR with Signed Commits**
2. Click "Run workflow"
3. Optionally enter a custom message
4. Click "Run workflow"

The workflow will:
1. Generate a token from your GitHub App
2. Make a change to `updates.txt`
3. Create a pull request with signed commits
4. The commits will show as "Verified" because they are signed by the GitHub App

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

## Notes

- When `sign-commits: true` is used, the `committer` and `author` inputs are ignored
- Commit signing only works with bot-generated tokens (GITHUB_TOKEN or GitHub App tokens), not with Personal Access Tokens (PATs)
- The GitHub API has a 40MiB limit for git blobs when signing commits

## References

- [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request)
- [Commit signature verification for bots](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#commit-signature-verification-for-bots)
- [Authenticating with GitHub App tokens](https://github.com/peter-evans/create-pull-request/blob/main/docs/concepts-guidelines.md#authenticating-with-github-app-generated-tokens)
- [actions/create-github-app-token](https://github.com/actions/create-github-app-token)