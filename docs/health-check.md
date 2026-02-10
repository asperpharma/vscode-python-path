# Health Check

This document describes how to monitor and verify the health of the Python Path VS Code extension.

## Overview

Health checks ensure the extension is functioning correctly in production environments. This includes monitoring activation, command registration, performance, and error tracking.

## Extension Activation Monitoring

### Activation Events

The extension activates on the following events (defined in `package.json`):

- `onLanguage:python` - When a Python file is opened
- `onCommand:extension.copyPythonPath` - When the Copy Python Path command is invoked

**Note:** The `generateImportStatement` activation event is listed in `package.json` but the functionality is handled by the `copyPythonPath` command based on text selection.

### Verification Steps

1. **Check activation on Python file open:**
   ```
   1. Open VS Code
   2. Open a .py file
   3. Verify the extension loads (check Extensions view)
   4. Check Output panel > Extension Host for any errors
   ```

2. **Verify activation events:**
   ```javascript
   // Check in VS Code Developer Console (Help > Toggle Developer Tools)
   vscode.extensions.getExtension('mgesbert.python-path').isActive
   // Should return: true
   ```

3. **Check extension logs:**
   - Open Command Palette (`Cmd+Shift+P` or `Ctrl+Shift+P`)
   - Run "Developer: Show Logs"
   - Select "Extension Host"
   - Look for activation messages or errors

## Command Registration Verification

### Available Commands

The extension registers the following command:

- `extension.copyPythonPath` - Copy Python Path / Generate Import Statement

This single command handles both basic path copying and import statement generation based on whether text is selected in the editor.

### Manual Verification

1. **Via Command Palette:**
   ```
   1. Open a Python file
   2. Open Command Palette (Cmd+Shift+P or Ctrl+Shift+P)
   3. Type "Copy Python Path"
   4. Verify command appears and executes without errors
   ```

2. **Via Context Menus:**
   - **Explorer Context Menu:**
     1. Right-click a .py file in Explorer
     2. Verify "Copy Python Path" appears in the menu
     3. Click and verify it copies the path to clipboard

   - **Editor Context Menu:**
     1. Right-click in a Python file editor
     2. Verify "Copy Python Path" appears in the menu
     3. Click and verify functionality

   - **Editor Title Context Menu:**
     1. Right-click the tab of a Python file
     2. Verify "Copy Python Path" appears in the menu
     3. Click and verify functionality

### Automated Verification

Add the following test to verify command registration:

```javascript
const vscode = require('vscode');
const assert = require('assert');

suite('Health Check Tests', function() {
  test('Commands are registered', async function() {
    const commands = await vscode.commands.getCommands(true);
    assert.ok(
      commands.includes('extension.copyPythonPath'),
      'copyPythonPath command should be registered'
    );
  });
});
```

## Functional Health Checks

### Test Scenarios

1. **Basic Path Copy:**
   ```
   File: /project/src/utils/helper.py
   Expected: utils.helper
   Action: Place cursor in file, run command
   Verify: Clipboard contains "utils.helper"
   ```

2. **Single Import Generation:**
   ```
   File: /project/src/utils/helper.py
   Selection: "my_function"
   Expected: from utils.helper import my_function
   Action: Select text, run command
   Verify: Clipboard contains correct import
   ```

3. **Multiple Import Generation:**
   ```
   File: /project/src/utils/helper.py
   Selections: "func1", "func2", "func3"
   Expected:
   from utils.helper import (
       func1,
       func2,
       func3,
   )
   Action: Multi-select text, run command
   Verify: Clipboard contains formatted import
   ```

4. **Edge Cases:**
   - Non-Python files (should not activate)
   - `__init__.py` files (should use parent module path)
   - Files outside Python packages (should use filename only)
   - Empty selections (should copy module path only)

### Automated Testing

Run the test suite:

```bash
npm test
```

The tests in `test/` directory verify core functionality.

## Performance Metrics

### Key Metrics to Monitor

1. **Activation Time:**
   - Target: < 100ms
   - Measure: Time from activation event to command registration
   - Check: Enable extension timing in VS Code settings

2. **Command Execution Time:**
   - Target: < 50ms
   - Measure: Time from command invocation to completion
   - Monitor: VS Code Developer Tools console

3. **Memory Usage:**
   - Target: < 10MB at rest
   - Monitor: VS Code Process Explorer (Help > Process Explorer)
   - Look for: "Extension Host (mgesbert.python-path)"

### Performance Testing

```javascript
// Add to test suite for performance monitoring
suite('Performance Tests', function() {
  test('Command execution completes quickly', async function() {
    const start = Date.now();
    await vscode.commands.executeCommand('extension.copyPythonPath');
    const duration = Date.now() - start;
    assert.ok(duration < 100, `Command took ${duration}ms (target: <100ms)`);
  });
});
```

## Error Tracking

### Common Errors

1. **File Not Found:**
   - Cause: Trying to copy path from non-existent file
   - Detection: Check console logs for file system errors
   - Resolution: Ensure file exists before command execution

2. **Invalid Python File:**
   - Cause: File doesn't end with .py
   - Detection: Empty clipboard or no action
   - Resolution: Verify file extension check in `getPythonPath()`

3. **Missing __init__.py:**
   - Cause: Package structure not properly detected
   - Detection: Path shorter than expected
   - Resolution: Verify __init__.py exists in parent directories

### Error Logging

The extension logs errors to the console:

```javascript
try {
  // Extension logic
} catch (e) {
  console.log(e);  // Current error logging
}
```

**Improvement Recommendation:**
Consider using VS Code's OutputChannel for better user visibility:

```javascript
const outputChannel = vscode.window.createOutputChannel('Python Path');

function logError(error) {
  outputChannel.appendLine(`[ERROR] ${new Date().toISOString()}: ${error.message}`);
  outputChannel.show();
}
```

### Error Monitoring

1. **Manual Monitoring:**
   - Check Developer Console (Help > Toggle Developer Tools)
   - Review Extension Host logs
   - Look for stack traces or error messages

2. **User Reports:**
   - Monitor GitHub Issues
   - Review VS Code Marketplace ratings/reviews
   - Check for error-related feedback

## User Feedback Collection

### Marketplace Reviews

Regularly monitor:
- Star ratings on VS Code Marketplace
- User reviews and comments
- Installation/uninstallation trends

### GitHub Issues

Track:
- Open issues count (target: < 5 open bugs)
- Issue resolution time (target: < 7 days for critical bugs)
- Issue categories (bugs, features, questions)

### Analytics (Optional)

If telemetry is enabled (requires user consent):

```javascript
const telemetry = vscode.env.telemetryEnabled;
if (telemetry) {
  // Track command usage
  // Track error rates
  // Track performance metrics
}
```

**Note:** Currently, the extension does not implement telemetry. Consider adding it with proper user consent.

## Health Check Checklist

Use this checklist for regular health checks:

### Daily/Automated Checks
- [ ] Extension activates on Python files
- [ ] Commands are registered successfully
- [ ] Automated tests pass
- [ ] No errors in extension logs
- [ ] Performance metrics within targets

### Weekly Checks
- [ ] Review new GitHub issues
- [ ] Check Marketplace reviews
- [ ] Verify installation counts are stable/growing
- [ ] Test on latest VS Code version
- [ ] Run full test suite on multiple platforms

### Monthly Checks
- [ ] Dependency vulnerability scan (`npm audit`)
- [ ] Update dependencies if needed
- [ ] Review and respond to user feedback
- [ ] Performance benchmark comparison
- [ ] Compatibility check with Python ecosystem updates

### Pre-Release Checks
- [ ] All automated tests pass
- [ ] Manual testing on Windows, macOS, and Linux
- [ ] Verify all commands work from all entry points
- [ ] Check performance metrics
- [ ] Review and update documentation
- [ ] Verify package contents with `vsce ls`

## Monitoring Tools

### Built-in VS Code Tools

1. **Extension Host Logs:**
   ```
   Command Palette > Developer: Show Logs > Extension Host
   ```

2. **Process Explorer:**
   ```
   Help > Process Explorer
   Look for: Extension Host (mgesbert.python-path)
   ```

3. **Developer Console:**
   ```
   Help > Toggle Developer Tools
   Check Console tab for errors
   ```

### External Tools

1. **npm audit:**
   ```bash
   npm audit
   npm audit fix
   ```

2. **VS Code Extension Test Runner:**
   ```bash
   npm test
   ```

3. **Code Coverage:**
   ```bash
   npm test -- --coverage
   ```

## Alerting and Notifications

### GitHub Actions (Recommended)

Set up automated health checks with GitHub Actions:

```yaml
name: Health Check

on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  workflow_dispatch:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm install
      - run: npm test
      - name: Report failure
        if: failure()
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Health Check Failed',
              body: 'Automated health check failed. Please investigate.',
              labels: ['bug', 'health-check']
            })
```

### Marketplace Monitoring

Set up notifications for:
- New reviews (daily check)
- Installation count drops (weekly check)
- Support requests (real-time if possible)

## Incident Response

### When Issues Are Detected

1. **Assess Severity:**
   - Critical: Extension completely broken
   - High: Major functionality broken
   - Medium: Minor feature broken
   - Low: Cosmetic or edge case issue

2. **Immediate Actions:**
   - Log the issue in GitHub
   - Reproduce the problem
   - Identify affected versions
   - Determine scope of impact

3. **Resolution:**
   - Fix the issue
   - Test thoroughly
   - Deploy hotfix (patch version)
   - Notify users if widespread

4. **Post-Incident:**
   - Document root cause
   - Add regression tests
   - Update health check procedures
   - Review prevention measures

## Best Practices

1. **Regular Testing:**
   - Run tests before every commit
   - Test on multiple VS Code versions
   - Test on different operating systems

2. **Proactive Monitoring:**
   - Don't wait for user reports
   - Set up automated checks
   - Review logs regularly

3. **Documentation:**
   - Keep health check procedures updated
   - Document known issues
   - Maintain runbooks for common problems

4. **User Communication:**
   - Respond to issues promptly
   - Provide clear error messages
   - Update changelog regularly

## References

- [Extension Testing - VS Code Documentation](https://code.visualstudio.com/api/working-with-extensions/testing-extension)
- [Extension Activation Events](https://code.visualstudio.com/api/references/activation-events)
- [Extension Capabilities](https://code.visualstudio.com/api/extension-capabilities/overview)
