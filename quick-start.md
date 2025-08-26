---
layout: page
title: Quick Start Guide
permalink: /quick-start/
---

# 🚀 Quick Start Guide

## Getting Started with GitHub Pages Automation

This guide will help you implement the comprehensive automation suite in your own GitHub Pages project.

## 📎 Prerequisites

- GitHub repository for your project
- Basic understanding of Git and GitHub
- GitHub Pages enabled on your repository

## 🛠️ Step 1: Repository Setup

### Enable GitHub Pages
1. Go to your repository **Settings**
2. Navigate to **Pages** section
3. Set **Source** to "GitHub Actions"
4. Save the settings

## 📁 Step 2: Add Workflow Files

Create the following directory structure in your repository:

```
.github/
├── workflows/
│   ├── deploy-jekyll.yml
│   ├── security.yml
│   ├── maintenance.yml
│   └── test-pr.yml
└── mlc_config.json
```

### Core Workflow Files

#### 1. Jekyll Deployment (`deploy-jekyll.yml`)

```yaml
name: 🚀 Deploy Jekyll Site

on:
  push:
    branches: [ main, master ]
    paths:
      - '_posts/**'
      - '_pages/**'
      - '_layouts/**'
      - '_includes/**'
      - '_sass/**'
      - 'assets/**'
      - '_config.yml'
      - 'Gemfile*'
      - 'index.md'
      - 'index.html'
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'production'
        type: choice
        options:
          - production
          - staging
          - development

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

env:
  RUBY_VERSION: '3.2'
  NODE_VERSION: '20'

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: 🔧 Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: ${{ env.RUBY_VERSION }}
          bundler-cache: true

      - name: 🔧 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: 📦 Install tools
        run: |
          npm install -g imagemin-cli terser cssnano-cli

      - name: 🔧 Setup Pages
        uses: actions/configure-pages@v4

      - name: 🏗️ Build Jekyll site
        run: |
          bundle exec jekyll build --verbose
        env:
          JEKYLL_ENV: ${{ github.event.inputs.environment || 'production' }}

      - name: 📤 Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: _site/

      - name: 🚀 Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

#### 2. Security Scanning (`security.yml`)

```yaml
name: 🔒 Security Scan

on:
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM UTC
  push:
    branches: [ main ]
    paths:
      - 'Gemfile*'
      - 'package*.json'
  workflow_dispatch:

permissions:
  contents: read
  security-events: write

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: 🔍 Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: 📤 Upload scan results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'
```

#### 3. Maintenance (`maintenance.yml`)

```yaml
name: 🔧 Maintenance

on:
  schedule:
    - cron: '0 3 * * *'  # Daily at 3 AM UTC
  workflow_dispatch:

permissions:
  contents: read

jobs:
  maintenance:
    runs-on: ubuntu-latest
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: 🔗 Check links
        uses: gaurav-nelson/github-action-markdown-link-check@v1
        with:
          use-quiet-mode: 'yes'
          config-file: '.github/mlc_config.json'
```

### Configuration Files

#### Link Checker Config (`.github/mlc_config.json`)

```json
{
  "ignorePatterns": [
    {"pattern": "^https://example\\.com"},
    {"pattern": "^mailto:"},
    {"pattern": "^tel:"},
    {"pattern": "^#"}
  ],
  "timeout": "20s",
  "retryOn429": true,
  "retryCount": 3
}
```

## 📝 Step 3: Jekyll Configuration

Ensure your `_config.yml` includes:

```yaml
# Site settings
title: Your Site Title
description: Your site description
url: "https://yourusername.github.io"
baseurl: "/your-repo-name"

# Build settings
markdown: kramdown
theme: minima

# Plugins
plugins:
  - jekyll-feed
  - jekyll-sitemap
  - jekyll-seo-tag

# Exclude from build
exclude:
  - node_modules
  - vendor
  - README.md
  - Gemfile
  - Gemfile.lock
```

## 🔄 Step 4: Test the Setup

1. **Commit and push** your changes
2. **Check the Actions tab** in your repository
3. **Monitor workflow execution**
4. **Visit your GitHub Pages URL** once deployed

## 🎉 You're Done!

Your GitHub Pages site now has:

- ✅ **Automated deployment** on every push
- ✅ **Daily security scans** 
- ✅ **Link validation**
- ✅ **Performance optimization**

## 📚 Next Steps

- [View all workflow examples](https://github.com/Clubhouse1661/github-pages-automation-demo/tree/main/.github/workflows)
- [Read the complete documentation]({{ '/workflow-documentation/' | relative_url }})
- [Troubleshooting guide]({{ '/troubleshooting/' | relative_url }})

---

### 📞 Need Help?

Check out the [troubleshooting guide]({{ '/troubleshooting/' | relative_url }}) or review the [complete workflow documentation]({{ '/workflow-documentation/' | relative_url }}).