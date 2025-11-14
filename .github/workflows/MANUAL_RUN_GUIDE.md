# How to Manually Run the Catalog Test Workflow

## Using GitHub UI (Easiest)

1. **Go to the Actions tab** in your GitHub repository
   - Navigate to: `https://github.com/YOUR_USERNAME/release-service-utils/actions`

2. **Select the workflow**
   - Click on "Test Against Release Service Catalog" in the left sidebar

3. **Click "Run workflow"** button (top right)
   
4. **Fill in the parameters:**

   ![Workflow Dispatch Screenshot](https://docs.github.com/assets/cb-32937/mw-1440/images/help/actions/workflow-dispatch-button.webp)

   - **Use workflow from**: Choose your branch (e.g., `main`, `my-feature-branch`)
   
   - **Catalog repository (owner/repo)**: 
     - Default: `konflux-ci/release-service-catalog`
     - **For your fork**: `YOUR_USERNAME/release-service-catalog`
   
   - **Catalog branch to test against**:
     - Default: `development`
     - **For your fork**: `main` or whatever branch you want
   
   - **Utils image to test**:
     - Default: (empty - uses PR image from Konflux)
     - **For manual testing**: `quay.io/YOUR_USERNAME/release-service-utils:your-tag`

5. **Click "Run workflow"**

## Example Configurations

### Test Your Fork Against Your Fork
```
Catalog repository: swickers/release-service-catalog
Catalog branch: development
Utils image: (leave empty to build from branch, or specify your test image)
```

### Test Custom Image Against Upstream
```
Catalog repository: konflux-ci/release-service-catalog
Catalog branch: development
Utils image: quay.io/swickers/release-service-utils:test-123
```

### Test Latest Main Against Your Fork
```
Catalog repository: swickers/release-service-catalog
Catalog branch: main
Utils image: quay.io/konflux-ci/release-service-utils:latest
```

## Using GitHub CLI

If you prefer command line:

```bash
# Test with your fork
gh workflow run test-against-catalog.yaml \
  -f catalog_repo=swickers/release-service-catalog \
  -f catalog_branch=development

# Test with custom image
gh workflow run test-against-catalog.yaml \
  -f catalog_repo=swickers/release-service-catalog \
  -f catalog_branch=main \
  -f utils_image=quay.io/swickers/release-service-utils:pr-123

# Check status
gh run list --workflow=test-against-catalog.yaml

# View logs
gh run view --log
```

## Using API

```bash
# Get the workflow ID
WORKFLOW_ID=$(gh api repos/YOUR_USERNAME/release-service-utils/actions/workflows \
  | jq -r '.workflows[] | select(.name == "Test Against Release Service Catalog") | .id')

# Trigger the workflow
gh api \
  --method POST \
  -H "Accept: application/vnd.github+json" \
  repos/YOUR_USERNAME/release-service-utils/actions/workflows/${WORKFLOW_ID}/dispatches \
  -f ref='main' \
  -f inputs[catalog_repo]='swickers/release-service-catalog' \
  -f inputs[catalog_branch]='development' \
  -f inputs[utils_image]=''
```

## What Happens Next

1. ✅ Workflow checks out both repos
2. ✅ Replaces all utils images in catalog
3. ✅ Creates a test PR in the catalog repo
4. ✅ Monitors the CI on that PR
5. ✅ Reports results
6. ✅ Auto-closes the test PR

## Tips

- **Leave `utils_image` empty** if running from a PR - it will automatically use the PR's built image
- **Specify `utils_image`** when testing a specific image you've already built
- **Use your fork** first to test the workflow itself before running against upstream
- **Check Actions logs** for detailed output if something fails

## Troubleshooting

**"Can't find workflow"**
- Make sure you've pushed the workflow file to your branch
- Refresh the Actions page

**"Permission denied creating PR"**
- You need a `CATALOG_PR_TOKEN` secret with write access to the catalog repo
- Or use your own fork where `GITHUB_TOKEN` has access

**"No image found"**
- If you left `utils_image` empty and triggered manually (not from PR), provide an image
- The workflow needs either a PR context or a manual image specified

