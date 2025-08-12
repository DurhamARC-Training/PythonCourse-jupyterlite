# DurhamARC JupyterLite deployment repository

This repository is not meant to be deployed on its own but the content is supposed to be filled by the notebooks and files provided in the repository. 

For this to work the Course repository needs to contain a pip installable  `requirements.txt` with all non-jupyter dependencies.

Paste this into a `.github/workflows/deploy.yml` within the course repository and activate Github Pages generated from Github Actions.

```
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