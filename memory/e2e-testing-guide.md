# E2E Testing Guide

Project-specific knowledge base for generating and running Playwright E2E tests.

> **This is a template.** After `fork-init`, customize this file in `.specify/memory/e2e-testing-guide.md` with your project's specific DOM patterns, page objects, CSS values, and pitfalls.

## App Structure

| Component | Detail |
|-----------|--------|
| **Frontend** | Port ??? — `E2E_BASE_URL` env var |
| **API** | Port ??? |
| **Auth** | Describe auth mechanism (Keycloak, Auth0, etc.) |
| **Config** | Path to `.env.e2e` (credentials, test data) |
| **Test runner** | `npx playwright test tests/{file}.spec.js --project=e2e` |
| **Config file** | Path to `playwright.config.js` |

### Environment Variables

```
E2E_USERNAME / E2E_PASSWORD  — Auth credentials
E2E_BASE_URL                 — App base URL
E2E_PAUSE                    — Set to "1" for page.pause() in tests
# Add project-specific env vars here
```

## DOM Patterns

Document selectors for common UI elements in your app:

### Framework-Specific Selectors
<!-- Example for MUI:
- **Autocomplete root:** `.MuiAutocomplete-root:has-text("Label") input`
- **Popover menus:** `.MuiPopover-paper`
- **Backdrop (loading):** `.MuiBackdrop-root`
-->

### App-Specific Selectors
<!-- Document custom component selectors here -->

## Highlight / Focus CSS Values

Document any CSS values used for focus/highlight states:

<!-- Example:
### Active Item (blue theme)
- `backgroundColor`: `rgb(227, 242, 253)` (`#e3f2fd`)
- `boxShadow`: contains `25, 118, 210`
-->

## Page Object Inventory

List all page objects with their methods:

<!-- Example:
### LoginPage (`login.page.js`)
| Method | Description |
|--------|-------------|
| `login(user, pass)` | Fill credentials and submit |
| `expectLoggedIn()` | Assert redirect to dashboard |
-->

## Assertion Helpers

List custom assertion functions:

<!-- Example:
| Function | Checks |
|----------|--------|
| `assertHighlighted(locator)` | backgroundColor + boxShadow |
-->

## Pitfalls & Lessons Learned

Document timing issues, flaky patterns, and workarounds:

### Timing
<!-- Example:
- Always add `waitForTimeout(300)` after dropdown interactions
- Wait for loading spinners to disappear before assertions
-->

### Known Issues
<!-- Example:
- Save button triggers backend call that may timeout without backend running
- Auth state expires after 30 minutes — re-run auth setup if tests fail with 401
-->

### Test Conventions
<!-- Example:
- Test files: `client/e2e/tests/{feature}.spec.js`
- Grouping: `test.describe('TICKET-ID: Feature Name', () => { ... })`
- Naming: `test('US1-AC1: description', ...)`
- Skip: `test.skip(condition, 'reason')` for unmet preconditions
- Debug: `if (process.env.E2E_PAUSE) await page.pause();`
-->
