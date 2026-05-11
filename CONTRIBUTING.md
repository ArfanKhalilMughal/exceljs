# Contributing to @zklogic/exceljs

Thank you for your interest in contributing to this project! This document provides guidelines for various ways to contribute and, importantly, the correct process for publishing releases.

## Reporting Issues

When reporting issues, please include:
- A minimal code example that reproduces the issue
- The version of exceljs you're using
- Your environment (Node.js version, browser, OS)
- Expected vs. actual behavior

## Submitting Pull Requests

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes and add tests
4. Run the test suite: `npm test`
5. Commit with clear messages following conventional commits
6. Push to your fork and create a Pull Request

## Publishing a Release

**⚠️ IMPORTANT: Always follow this exact process to ensure proper packaging**

### Prerequisites
- You must have publish access to the npm package
- All changes must be committed and pushed to the repository

### Release Process

```bash
# 1. Ensure you're on the development or main branch
git checkout development

# 2. Ensure you have the latest changes
git pull origin development

# 3. Ensure workspace is clean
git status

# 4. Update the version number
# This will trigger the complete release workflow:
npm version [major|minor|patch]

# Examples:
# npm version patch   # 5.0.0 → 5.0.1
# npm version minor   # 5.0.0 → 5.1.0
# npm version major   # 5.0.0 → 6.0.0
```

### What `npm version` does automatically:

1. **preversion hook** (before version change):
   - Cleans build artifacts: `npm run clean`
   - Builds the dist directory: `npm run build`
   - Runs comprehensive tests: `npm run test:version`
   - If any step fails, the whole process is aborted

2. **Version update**: Updates version in package.json and package-lock.json

3. **Git commit**: Creates a commit with message like "5.0.1"

4. **Git tag**: Creates a git tag like "v5.0.1"

5. **postversion hook** (after version change):
   - Pushes commits: `git push --no-verify`
   - Pushes tags: `git push --tags --no-verify`

6. **GitHub Actions trigger**:
   - The push tag triggers `.github/workflows/tests.yml`
   - The publish job runs the full test suite again
   - Then builds and publishes to npm

### Do NOT do this:

❌ **WRONG - Never use these approaches:**

```bash
# DON'T do this:
npm publish

# DON'T do this:
git tag v5.0.1
npm publish

# DON'T do this:
git commit -m "5.0.1"
npm publish

# DON'T do this (skips preversion hooks):
npm version patch --no-git-tag-version
npm publish
```

### Verification Steps

After running `npm version patch`, verify:

1. **Local changes**:
   ```bash
   git log --oneline -5          # Should show new commit
   git tag | tail -5             # Should show new tag
   ls dist/exceljs.min.js        # File should exist
   ```

2. **CI/CD Status**:
   - Visit GitHub Actions workflow
   - Verify the publish job completed successfully
   - Check npm registry: https://www.npmjs.com/package/@zklogic/exceljs
   - Verify dist/ directory is present in the tarball

3. **Test the published package**:
   ```bash
   npm install @zklogic/exceljs@latest
   # or test in a new project:
   mkdir test-package && cd test-package
   npm init -y
   npm install @zklogic/exceljs
   ```

## Build System

The project uses Grunt for building:

- `npm run build`: Creates dist/ directory with browser bundles
- `npm run clean`: Removes build/dist artifacts
- `npm run test`: Runs all test suites
- `npm run lint`: Checks code style

## Safety Mechanisms

The project includes several safety mechanisms to prevent broken releases:

1. **preversion hook**: Ensures build succeeds before version change
2. **prepublishonly script**: Validates dist/ exists before npm publish
3. **GitHub Actions verification**: Rebuilds and tests before publishing
4. **Package.json validation**: dist/ is in the files array for npm inclusion

## Troubleshooting

### "dist/ directory is empty or missing" error

If you see this error when running `npm publish`:

```bash
# The build artifact is missing, rebuild it:
npm run build

# Then try again:
npm publish
```

### Git push fails during npm version

If the postversion hook fails to push:

```bash
# Manually push the changes:
git push origin development
git push --tags
```

### GitHub Actions publish failed

Check the workflow logs:
1. Go to Actions > Tests workflow
2. Click the failed run
3. Check the publish job logs
4. Common issues:
   - Build failed (check npm run build output)
   - Tests failed (check test output)
   - npm token missing or invalid

## Code Style

- Follow ESLint rules: `npm run lint`
- Auto-fix most issues: `npm run lint:fix`
- Code style is enforced via git hook (husky)

## Testing

```bash
npm run test              # Run all tests
npm run test:unit        # Unit tests only
npm run test:integration # Integration tests only
npm run test:end-to-end  # End-to-end tests
npm run test:version     # Full suite (same as preversion)
```

## Performance

Before committing large changes:

```bash
npm run benchmark
# This measures performance impact on key operations
```

## Additional Resources

- [Model Documentation](./MODEL.md)
- [Release Notes](./UPGRADE-4.0.md)
- [Migration Guide](./MIGRATION_EXECUTION_SUMMARY.md)
- [Original Repository](https://github.com/exceljs/exceljs)

## Questions?

If you have questions about the contribution process or release procedure, please refer to:
- `PUBLISH_FAILURE_ANALYSIS.md` - Details about the v5.0.0 failure and prevention
- `.github/workflows/tests.yml` - The automated CI/CD workflow
- `gruntfile.js` - The build configuration

