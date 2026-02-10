# Deployment Flow

This document describes the deployment process for the Python Path VS Code extension.

## Overview

The Python Path extension is published to the Visual Studio Code Marketplace. The deployment flow involves building, testing, packaging, and publishing the extension.

## The Launch Trigger: Deploy to Production

Your code is ready. You just need to push the "Start" button.

### Automated Deployment

The repository is configured with GitHub Actions to automatically deploy the extension to production when changes are merged to the main branch.

**Action:** Merge your changes to the main branch on GitHub.

**Methods:**
- Click "Merge Pull Request" in the GitHub UI
- Or use the command line:
  ```bash
  git checkout main
  git merge your-feature-branch
  git push origin main
  ```

**Result:** This signals GitHub Actions to automatically:
1. Run all tests to ensure code quality
2. Build and package the extension
3. Publish the new version to the VS Code Marketplace
4. Create a GitHub release with the packaged `.vsix` file

**Prerequisites for Automated Deployment:**
- The `VSCE_PAT` secret must be configured in the repository settings
- All tests must pass
- The version in `package.json` must be higher than the currently published version

**Monitoring the Deployment:**
- Navigate to the "Actions" tab in the GitHub repository
- Find the workflow run triggered by your merge
- Monitor the deployment progress and check for any errors
- Once complete, verify the new version on the VS Code Marketplace

**Manual Override:**
If you need to trigger the deployment workflow manually without merging to main, you can use the "workflow_dispatch" event:
- Go to Actions tab in GitHub
- Select "Deploy to Production" workflow
- Click "Run workflow"
- Choose the branch and click "Run workflow"

## Prerequisites

Before deploying, ensure you have:

- Node.js (v18 or later recommended, minimum v12) installed
- npm package manager
- Visual Studio Code Extension Manager (`vsce`) installed globally:
  ```bash
  npm install -g @vscode/vsce
  ```
- Publisher account on VS Code Marketplace
- Personal Access Token (PAT) with Marketplace publishing permissions

## Local Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/asperpharma/vscode-python-path.git
   cd vscode-python-path
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Test locally:**
   ```bash
   npm test
   ```

4. **Run in development mode:**
   - Open the project in VS Code
   - Press `F5` to launch the Extension Development Host
   - Test the extension features in the development window

## Build Process

### 1. Update Version

Update the version in `package.json` following semantic versioning:

```json
{
  "version": "X.Y.Z"
}
```

- **X (Major)**: Breaking changes
- **Y (Minor)**: New features, backward compatible
- **Z (Patch)**: Bug fixes, backward compatible

### 2. Update Changelog

Document changes in `CHANGELOG.md`:

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New features

### Changed
- Modified functionality

### Fixed
- Bug fixes
```

### 3. Validate Package

Ensure the extension package is valid:

```bash
vsce ls
```

This lists all files that will be included in the package.

### 4. Package the Extension

Create a `.vsix` package file:

```bash
vsce package
```

This creates a file named `python-path-X.Y.Z.vsix`.

## Testing the Package

Before publishing, test the packaged extension:

1. **Install the package locally:**
   ```bash
   code --install-extension python-path-X.Y.Z.vsix
   ```

2. **Test all features:**
   - Copy Python Path from command palette
   - Copy Python Path from explorer context menu
   - Copy Python Path from editor context menu
   - Generate import statements with single selection
   - Generate import statements with multiple selections

3. **Uninstall after testing:**
   ```bash
   code --uninstall-extension mgesbert.python-path
   ```

## Publishing to Marketplace

### First-Time Setup

If this is your first time publishing:

1. Create a publisher account at https://marketplace.visualstudio.com/manage
2. Create a Personal Access Token (PAT) with Marketplace scope
3. Login using vsce:
   ```bash
   vsce login <publisher-name>
   ```

### Publishing a New Version

1. **Publish the extension:**
   ```bash
   vsce publish
   ```

   Or specify the version increment type:
   ```bash
   vsce publish patch  # Increments Z
   vsce publish minor  # Increments Y
   vsce publish major  # Increments X
   ```

2. **Verify publication:**
   - Visit the extension page: https://marketplace.visualstudio.com/items?itemName=mgesbert.python-path
   - Check that the new version is live
   - Verify the changelog and README are displayed correctly

### Publishing from CI/CD

For automated deployments:

1. **Store PAT as secret** in your CI/CD system
2. **Use the following commands in your workflow:**
   ```bash
   npm install -g @vscode/vsce
   vsce publish -p $VSCE_PAT
   ```

### GitHub Actions Automated Deployment

The repository includes a GitHub Actions workflow (`.github/workflows/deploy.yml`) that automatically deploys the extension when changes are merged to the main branch.

**Setup Steps:**

1. **Configure the VSCE_PAT secret:**
   - Generate a Personal Access Token (PAT) from Azure DevOps with Marketplace publishing permissions
   - Go to your GitHub repository > Settings > Secrets and variables > Actions
   - Click "New repository secret"
   - Name: `VSCE_PAT`
   - Value: Your Personal Access Token
   - Click "Add secret"

2. **Workflow Triggers:**
   - **Automatic:** Triggers on push to `main` branch
   - **Manual:** Can be triggered manually via the Actions tab using workflow_dispatch

3. **Workflow Steps:**
   - Checks out the code
   - Sets up Node.js environment
   - Installs dependencies
   - Runs tests
   - Packages the extension
   - Publishes to VS Code Marketplace
   - Creates a GitHub release with the packaged `.vsix` file

4. **Monitoring:**
   - View workflow runs in the Actions tab
   - Check for green checkmarks (success) or red X's (failure)
   - Review logs for any errors or issues

**Important Notes:**
- Ensure the version in `package.json` is incremented before merging to main
- All tests must pass for deployment to succeed
- The workflow will only publish if the push event occurs (not on manual workflow_dispatch without the publish step enabled)

## Version Management

### Pre-release Versions

To publish a pre-release version:

```bash
vsce publish --pre-release
```

### Unpublishing

To remove a version (not recommended):

```bash
vsce unpublish mgesbert.python-path@X.Y.Z
```

**Note**: Unpublishing can confuse users who have already installed the version. Prefer publishing a new patch version with fixes.

## Post-Deployment

After successful deployment:

1. **Create a Git tag:**
   ```bash
   git tag -a vX.Y.Z -m "Release version X.Y.Z"
   git push origin vX.Y.Z
   ```

2. **Create a GitHub release:**
   - Go to the repository's releases page
   - Create a new release from the tag
   - Add release notes from CHANGELOG.md
   - Attach the `.vsix` file to the release

3. **Monitor for issues:**
   - Check extension ratings and reviews
   - Monitor GitHub issues for bug reports
   - Review telemetry data (if enabled)

## Rollback Procedure

If a deployed version has critical issues:

1. **Publish a hotfix version (RECOMMENDED):**
   - The best approach is to create a new patch version with the fix
   - Update CHANGELOG.md to document the issue and fix
   - Test thoroughly before publishing
   - Publish the hotfix version (e.g., if 1.2.3 is broken, publish 1.2.4 with fixes)

   ```bash
   # After fixing the issue
   # Update version in package.json to 1.2.4
   # Update CHANGELOG.md
   vsce publish
   ```

2. **Alternative - Emergency rollback (use only if hotfix isn't immediately possible):**
   - Checkout the previous stable version from git
   - Increment to a new version number (higher than the broken one)
   - Update CHANGELOG.md to document the rollback:
     ```markdown
     ## [1.2.4] - YYYY-MM-DD
     ### Reverted
     - Rolled back changes from 1.2.3 due to critical bug
     - Restored functionality from version 1.2.2
     ```
   - Publish the rolled-back code as the new version

3. **Notify users** through:
   - GitHub issues
   - Extension changelog
   - Marketplace description update

4. **Post-rollback:**
   - Document the issue and root cause in GitHub issues
   - Add regression tests to prevent recurrence
   - Plan proper fix for next version

**Important:** Never attempt to unpublish or re-publish the same version number, as this can cause issues for users who have already installed it.

## Security Considerations

- **Never commit** the Personal Access Token to the repository
- **Use environment variables** or secure secret management for CI/CD
- **Review dependencies** regularly for security vulnerabilities:
  ```bash
  npm audit
  npm audit fix
  ```

## Troubleshooting

### Common Issues

1. **"Failed to publish" error:**
   - Verify your PAT is valid
   - Check that you have publisher permissions
   - Ensure version number is higher than current published version

2. **Missing files in package:**
   - Check `.vscodeignore` file
   - Use `vsce ls` to preview included files

3. **Extension not activating:**
   - Verify `activationEvents` in `package.json`
   - Check extension logs in VS Code developer console

## References

- [Publishing Extensions - VS Code Documentation](https://code.visualstudio.com/api/working-with-extensions/publishing-extension)
- [vsce - Publishing Tool Documentation](https://github.com/microsoft/vscode-vsce)
- [Extension Manifest Reference](https://code.visualstudio.com/api/references/extension-manifest)
