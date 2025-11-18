# JupyterLite Deployment Template

Deploy Jupyter notebooks as interactive JupyterLite sites on GitHub Pages with automatic branch deployments.

## Quick Start (2 Steps)

### 1. Create Workflow File

Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Deploy JupyterLite

on:
  push:
    branches: ['**']  # Deploy all branches

permissions:
  contents: write

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit
```

### 2. Enable GitHub Pages

After the first workflow run:

1. Go to **Settings** → **Pages**
2. Source: **Deploy from a branch**
3. Branch: **gh-pages** / **/ (root)**
4. Click **Save**

**Done!** Your notebooks are now deployed. The workflow automatically creates the `gh-pages` branch on first run.

## Key Features

- **Zero-config deployment** - works out of the box  
- **Branch deployments** - preview changes at `https://org.github.io/repo/branch/branch-name/`  
- **Auto-cleanup** - removes deployments when branches are deleted  
- **SEO protection** - branch deployments blocked from search engines  
- **Auto-updates** - improvements to template apply automatically  
- **No gh-pages setup** - branch created automatically on first run

## Branch Deployments

| Branch | Deployment URL | Indexed? |
|--------|---------------|----------|
| `main` | `https://org.github.io/repo/` | Yes |
| `feature-x` | `https://org.github.io/repo/branch/feature-x/` | No |
| `fix/bug-123` | `https://org.github.io/repo/branch/fix-bug-123/` | No |

- Branch names with `/` are converted to `-` in URLs
- Non-main branches are protected from search engines via `robots.txt` and meta tags

## Repository Requirements

**Required:**
- Jupyter notebooks (`.ipynb` files) anywhere in your repo

**Optional:**
- `requirements.txt` - Python packages for your notebooks

**Example structure:**
```
my-course/
├── .github/workflows/deploy.yml
├── notebooks/*.ipynb
├── requirements.txt  (optional)
└── data/*.csv  (optional)
```

## Configuration Options

All parameters are optional with sensible defaults:

```yaml
jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    with:
      python-version: '3.11'              # Python version (default: 3.11)
      deploy-branches: true               # Deploy non-main branches (default: true)
      cleanup-stale-branches: true        # Auto-cleanup (default: true)
      template-repo: 'Org/CustomTemplate' # Custom template (default: this repo)
    secrets: inherit
```

### Deploy Only Main Branch

```yaml
on:
  push:
    branches: [main]

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    with:
      deploy-branches: false
    secrets: inherit
```

### Deploy Specific Branch Patterns

```yaml
on:
  push:
    branches:
      - main
      - 'feature/**'
      - 'release/**'

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit
```

## Automatic Cleanup of Deleted Branches

**Cleanup happens automatically in two ways:**

### 1. On Every Push (Default - Enabled)

When you push to any branch, stale deployments are automatically removed:

```yaml
# Default behavior - cleanup is enabled
jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit
```

**Example:** Delete `feature-x` locally, push to `main` → `feature-x` deployment is removed.

To disable:
```yaml
with:
  cleanup-stale-branches: false
```

### 2. Immediate Cleanup on Branch Delete (Optional)

For instant cleanup when you delete a branch on GitHub:

```yaml
on:
  push:
    branches: ['**']
  delete:  # Add this trigger

jobs:
  deploy:
    if: github.event_name == 'push'
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit

  cleanup:
    if: github.event_name == 'delete'
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/cleanup-branch-deployment.yml@main
    secrets: inherit
```

This removes the deployment immediately without waiting for the next push.

## Troubleshooting

### Deployment Not Appearing

1. **Enable GitHub Pages**: Settings → Pages → Source: "Deploy from a branch" → Branch: "gh-pages" → Folder: "/ (root)"
2. **Check workflow permissions**: Settings → Actions → General → Workflow permissions: "Read and write permissions"
3. **Wait a few minutes** after first deployment for GitHub Pages to activate
4. **Check Actions logs** for errors

### Workflow Fails with "gh-pages does not exist"

The workflow now automatically creates the `gh-pages` branch on first run. If you still see this error, it may be a transient issue - re-run the workflow. Otherwise raise an issue here.

### Branch Deployment Shows 404

- First deployment to any branch creates the `gh-pages` branch
- Wait 2-3 minutes for GitHub Pages to update
- Check that the branch deployment exists at: `https://org.github.io/repo/branch/branch-name/`

### Custom Domain

Branch deployments work with custom domains:
- Main: `https://customdomain.com/`
- Branches: `https://customdomain.com/branch/feature-x/`

## Version Pinning

**Recommended:** Use `@main` for automatic updates:
```yaml
uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
```

**Stable:** Pin to a specific version:
```yaml
uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@v1.0.0
```

---

## Why Use This Template?

- **Automatic updates** - bug fixes and features apply to all repos  
- **No gh-pages setup** - branch created automatically  
- **Branch deployments** - preview changes before merging  
- **Auto-cleanup** - no stale deployments  
- **Consistent** - all repos use the same proven workflow  
