# DurhamARC JupyterLite deployment repository

This repository serves as a **template and reusable GitHub Action** for deploying Jupyter notebooks as JupyterLite sites on GitHub Pages.

## Quick Start

**Use the reusable action** - no need to copy YAML!

Create `.github/workflows/deploy.yml` in your content repository:

```yaml
name: Deploy JupyterLite

on:
  push:
    branches: ['**']

permissions:
  contents: write  # Required to push to gh-pages branch

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit
```

Then enable GitHub Pages in your repository:

1. Go to **Settings** → **Pages**
2. Under **Source**, select **Deploy from a branch**
3. Select **gh-pages** branch and **/ (root)** folder
4. Click **Save**

**Note:** The `gh-pages` branch will be created automatically on the first workflow run. If it doesn't exist yet, just run the workflow once and then configure Pages.

That's it! Now every push will trigger a deployment.

**Features:**

- Automatic deployment on push
- Branch deployments to `https://org.github.io/repo/branch/<sanitized-branch-name>/`
- Search engine protection for non-main branches (robots.txt + meta tags)
- Auto-updates when this template improves
- No YAML duplication across repositories

> **Note:** In deployment URLs, branch names with `/` are converted to `-` (e.g., `feature/my-branch` → `feature-my-branch`).

## How It Works

### Branch Deployment Behavior

| Branch | Deployment URL | Search Engine Indexing |
|--------|---------------|------------------------|
| `main` | `https://<org>.github.io/<repo>/` | Indexed (normal) |
| `feature-x` | `https://<org>.github.io/<repo>/branch/feature-x/` | Blocked (robots.txt + meta tags) |
| `fix/bug-123` | `https://<org>.github.io/<repo>/branch/fix-bug-123/` | Blocked |

**Note**: Branch names with `/` are converted to `-` in URLs (e.g., `fix/bug-123` → `fix-bug-123`)

### Search Engine Protection

For non-main branches, the action automatically:

1. **Creates `robots.txt`**:
   ```txt
   User-agent: *
   Disallow: /
   ```

2. **Adds meta tags** to all HTML files:
   ```html
   <meta name="robots" content="noindex, nofollow">
   ```

This prevents Google and other search engines from indexing branch deployments, ensuring only your main deployment appears in search results.

## Repository Requirements

Your content repository needs:

- Jupyter notebooks (`.ipynb` files) - place anywhere in the repo
- `requirements.txt` (optional) - Python packages needed by your notebooks

### Example Repository Structure

```
my-python-course/
├── .github/
│   └── workflows/
│       └── deploy.yml          # Deployment workflow
├── notebooks/
│   ├── 01-intro.ipynb
│   ├── 02-pandas.ipynb
│   └── 03-visualization.ipynb
├── data/
│   └── sample.csv              # Data files for notebooks
├── requirements.txt            # Python dependencies
└── README.md
```

### Example requirements.txt

```txt
numpy>=1.24.0
pandas>=2.0.0
matplotlib>=3.7.0
```

---

## Configuration Options

### Basic Configuration

```yaml
permissions:
  contents: write  # Required to push to gh-pages branch

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    with:
      python-version: '3.11'           # Optional: Python version (default: 3.11)
      deploy-branches: true             # Optional: Deploy non-main branches (default: true)
      cleanup-stale-branches: true      # Optional: Auto-cleanup deleted branches (default: true)
    secrets: inherit
```

### Deploy Only Main Branch

If you don't want branch deployments:

```yaml
name: Deploy JupyterLite

on:
  push:
    branches: [main]  # Only trigger on main

permissions:
  contents: write  # Required to push to gh-pages branch

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    with:
      deploy-branches: false
    secrets: inherit
```

### Custom Template Repository

If you fork this template:

```yaml
permissions:
  contents: write  # Required to push to gh-pages branch

jobs:
  deploy:
    uses: YourOrg/YourTemplate/.github/workflows/deploy-jupyterlite.yml@main
    with:
      template-repo: 'YourOrg/YourTemplate'
    secrets: inherit
```

---

## Examples

### Example 1: Deploy Main + Feature Branches

**Workflow**: `.github/workflows/deploy.yml`

```yaml
name: Deploy JupyterLite

on:
  push:
    branches:
      - main
      - 'feature/**'
      - 'fix/**'

permissions:
  contents: write  # Required to push to gh-pages branch

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit
```

**Result**:

- `main` → `https://org.github.io/repo/`
- `feature/auth` → `https://org.github.io/repo/branch/feature-auth/` (noindex)
- `fix/typo` → `https://org.github.io/repo/branch/fix-typo/` (noindex)

### Example 2: Preview Deployments for Pull Requests

```yaml
name: Deploy JupyterLite

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: write  # Required to push to gh-pages branch

jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    secrets: inherit
```

When you open a PR from `feature-branch`:

- Deployment URL: `https://org.github.io/repo/branch/feature-branch/`
- Share the URL with reviewers to preview changes

---

## Accessing Branch Deployments

After pushing a branch, check the **Actions** tab:

1. Click on the workflow run
2. Look for the **deploy** job
3. The deployment URL is shown in the job summary
4. Visit `https://<org>.github.io/<repo>/branch/<branch-name>/`

---

## Updating the Template

### Automatic Updates

When you use `@main` in the workflow:

```yaml
uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
```

Your deployments automatically use the latest version of the action.

### Pinning to a Specific Version

For stability, pin to a commit SHA or tag:

```yaml
uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@v1.0.0
```

Or a specific commit:

```yaml
uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@abc123def
```

---

## Troubleshooting

### Deployment Not Appearing

1. **Check GitHub Pages is enabled**: Settings → Pages → Source: "Deploy from a branch" → Branch: "gh-pages" → Folder: "/ (root)"
2. **Check workflow permissions**: Settings → Actions → General → Workflow permissions: "Read and write permissions"
3. **Check Actions logs**: Actions tab → Click failed/successful run → View logs
4. **Verify gh-pages branch exists**: After the first workflow run, check that the gh-pages branch was created

### Branch Deployment 404

The workflow automatically creates and manages the `gh-pages` branch:

1. **First deployment** (to any branch) creates the `gh-pages` branch automatically
2. Subsequent deployments update the branch with new content
3. Wait a few minutes for GitHub Pages to update after the first deployment

**Note:** The workflow handles the initial `gh-pages` branch creation automatically - no manual setup required!

### Search Engines Still Indexing Branches

1. Verify `robots.txt` exists: `https://org.github.io/repo/branch/feature-x/robots.txt`
2. Verify meta tags: View page source and check for `<meta name="robots" content="noindex, nofollow">`
3. Request removal from Google Search Console if already indexed

### Custom Domain Issues

If using a custom domain:

- Main branch works automatically
- Branch URLs: `https://customdomain.com/branch/feature-x/`
- Ensure your DNS settings support subdirectories

---

## Automatic Cleanup of Deleted Branch Deployments

**By default, stale branch deployments are automatically cleaned up** when you push to any branch. No additional configuration needed!

**Example:** If you delete the `feature-x` branch locally and push to any other branch (like `feature-y`), the deployment for `feature-x` at `https://org.github.io/repo/branch/feature-x/` will be automatically removed.

The reusable action includes two methods to automatically clean up deployments when branches are deleted:

### Method 1: Automatic Cleanup on Push (Default)

By default, every time a branch is deployed, the action checks for stale deployments and removes them. This means that when you push to any branch, deployments for deleted branches are automatically cleaned up.

**How it works:**

- Compares deployed branches in `gh-pages` with active branches in your repository
- Removes any deployments for branches that no longer exist
- Happens automatically on every branch deployment

**Disable if needed:**

```yaml
jobs:
  deploy:
    uses: DurhamARC-Training/PythonCourse-jupyterlite/.github/workflows/deploy-jupyterlite.yml@main
    with:
      cleanup-stale-branches: false  # Disable automatic cleanup
    secrets: inherit
```

### Method 2: Cleanup on Branch Delete Event

For immediate cleanup when a branch is deleted, add this to your workflow:

```yaml
name: Deploy JupyterLite

on:
  push:
    branches: ['**']
  delete:  # Trigger on branch deletion

permissions:
  contents: write  # Required to push to gh-pages branch

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

This will remove the deployment immediately when a branch is deleted, rather than waiting for the next branch push.

### Manual Cleanup

You can also manually clean up deployments:

```bash
# Clone gh-pages branch
git clone -b gh-pages https://github.com/org/repo.git
cd repo

# Remove old branch deployments
rm -rf branch/old-feature-name

# Commit and push
git add .
git commit -m "Clean up old branch deployment"
git push
```

---

## Benefits Over Copy-Paste Workflow

| Aspect | Copy-Paste | Reusable Action |
|--------|------------|-----------------|
| **Updates** | Manual in each repo | Automatic from template |
| **Maintenance** | Fix bugs in N repos | Fix once, all repos benefit |
| **Features** | Copy new features manually | Get new features automatically |
| **Branch deployments** | Not supported | Built-in |
| **SEO protection** | Manual implementation | Automatic |
| **Consistency** | Can drift apart | Always consistent |

---

## Documentation

See [WORKFLOW_GUIDE.md](WORKFLOW_GUIDE.md) for a detailed explanation of how the workflow works from scratch.

---

## Legacy: Copy-Paste Workflow

<details>
<summary>Click to expand the old copy-paste approach (not recommended)</summary>

Paste this into a `.github/workflows/deploy.yml` within the course repository and activate Github Pages generated from Github Actions.

```yaml
name: Deploy Jupyter Notebooks to GitHub Pages

on:
  push:
    branches: [ main ]

env:
  TEMPLATE_REPO: 'DurhamARC-Training/PythonCourse-jupyterlite'
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout template repository
      uses: actions/checkout@v4
      with:
        repository: ${{ env.TEMPLATE_REPO }}

    - name: Checkout main repository into content directory
      uses: actions/checkout@v4
      with:
        path: content

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.11'

    - name: Install template dependencies
      run: |
        python -m pip install -r requirements.txt

    - name: Install content dependencies
      run: |
        if [ -f content/requirements.txt ]; then
          echo "Found content/requirements.txt, installing dependencies..."
          python -m pip install -r content/requirements.txt
        else
          echo "No content/requirements.txt found, skipping content dependencies"
        fi

    - name: Build the JupyterLite site
      run: |
        jupyter lite build --contents content --output-dir dist

    - name: Upload artifact
      uses: actions/upload-pages-artifact@v3
      with:
        path: ./dist

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    permissions:
      pages: write
      id-token: write

    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}

    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Note:** This approach is deprecated. Use the reusable action instead for automatic updates and branch deployments.

</details>
