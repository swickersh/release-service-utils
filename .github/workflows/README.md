# GitHub Actions Workflows

## test-against-catalog.yaml

This workflow automatically tests release-service-utils PRs against the full release-service-catalog e2e test suite.

### What It Does

When you open or update a PR in release-service-utils, this workflow:

1. **Waits for Konflux** to build your PR image
2. **Checks out release-service-catalog** 
3. **Replaces all utils images** in the catalog with your PR image using `scripts/replace-utils-image.sh`
4. **Creates a test PR** in release-service-catalog
5. **Monitors the catalog CI** to see if all e2e tests pass
6. **Reports results** back to your utils PR
7. **Auto-closes** the test PR when done

### Benefits

- ✅ **Catch breaking changes early** - Know if your utils changes break any catalog tasks
- ✅ **Automated** - No manual testing needed
- ✅ **Fast feedback** - Get results while your PR is under review
- ✅ **Safe merging** - Only merge when catalog tests pass

### Setup Requirements

#### 1. GitHub Token (Required)

The workflow needs a GitHub token with permissions to create PRs in the catalog repo.

**Option A: Use a GitHub App (Recommended)**
- More secure with fine-grained permissions
- Better rate limits
- Can be restricted to specific repositories

**Option B: Use a Personal Access Token (PAT)**
1. Create a PAT with `repo` scope
2. Add it as a repository secret named `CATALOG_PR_TOKEN`

To add the secret:
```bash
# In the release-service-utils repository settings:
Settings → Secrets and variables → Actions → New repository secret

Name: CATALOG_PR_TOKEN
Value: <your-token>
```

**Without this token**, the workflow will fall back to `GITHUB_TOKEN` which may not have permission to create PRs in the catalog repo.

#### 2. Repository Access

The token must have write access to `konflux-ci/release-service-catalog` to:
- Create branches
- Create and close PRs
- Add comments and labels

### Workflow Diagram

```
PR Opened/Updated in release-service-utils
            ↓
    Wait for Konflux Build
            ↓
    Checkout Catalog Repo
            ↓
    Replace All Utils Images
            ↓
    Create Test PR in Catalog
            ↓
    Monitor Catalog CI
            ↓
    ┌─────────┴─────────┐
    ↓                   ↓
CI Passes          CI Fails
    ↓                   ↓
✅ Comment         ❌ Comment
on Utils PR        on Utils PR
    ↓                   ↓
    └─────────┬─────────┘
              ↓
    Close Catalog Test PR
```

### Example Usage

**Your PR workflow:**
```bash
1. Open PR in release-service-utils
2. ⏳ Wait for automated testing...
3. ✅ Get comment: "All catalog tests passed!"
4. ✔️  Merge with confidence
```

**Catalog test PR example:**
- Title: `test: utils PR #123 - Add new feature`
- Labels: `automated-test`, `do-not-merge`
- Body: Links back to original PR, shows image being tested
- Auto-closes after CI completes

### Customization

You can customize the workflow by editing `.github/workflows/test-against-catalog.yaml`:

- **Catalog repo**: Change `CATALOG_REPO` env var (default: `konflux-ci/release-service-catalog`)
- **Base branch**: Change `CATALOG_BRANCH` env var (default: `development`)
- **Timeouts**:
  - Konflux build: 30 minutes (1800s)
  - Catalog CI: 2 hours (7200s)
- **Poll intervals**: 30s for build, 60s for CI

### Troubleshooting

#### Workflow fails: "Timeout waiting for Konflux build"
- Check that the Konflux PR pipeline is configured correctly
- Verify the PR triggered a build in Konflux
- Increase `MAX_WAIT` in the "Wait for Konflux image build" step

#### Workflow fails: "Permission denied"
- Verify `CATALOG_PR_TOKEN` secret is set correctly
- Check token has `repo` scope
- Ensure token hasn't expired

#### Catalog PR isn't being created
- Check GitHub Actions logs for the "Create catalog PR" step
- Verify the replace-utils-image.sh script made changes
- Check for authentication issues

#### CI monitoring times out
- Catalog e2e tests may be taking longer than 2 hours
- Increase `MAX_WAIT` in the "Monitor catalog PR CI" step
- Check the catalog PR manually for stuck tests

### Manual Testing

If you need to test manually without the automation:

```bash
# Build your utils image
docker build -t quay.io/myuser/release-service-utils:test .

# Clone catalog
git clone https://github.com/konflux-ci/release-service-catalog.git
cd release-service-catalog

# Replace images
./scripts/replace-utils-image.sh quay.io/myuser/release-service-utils:test

# Create PR and run tests
git checkout -b test-my-changes
git add tasks/
git commit -m "test: my utils changes"
git push
gh pr create
```

### Disabling the Workflow

To temporarily disable this workflow:

1. Go to Actions tab in GitHub
2. Click on "Test Against Release Service Catalog" 
3. Click "..." → "Disable workflow"

Or add this to the workflow file:
```yaml
on:
  workflow_dispatch:  # Only allow manual triggers
```

### Future Enhancements

Potential improvements:
- [ ] Only run on specific label (e.g., `test-catalog`)
- [ ] Allow selecting which catalog branch to test against
- [ ] Cache between runs to speed up checkout
- [ ] Parallel testing across multiple catalog branches
- [ ] Integration with Konflux notifications

