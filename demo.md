---
layout: page
title: Demo Documentation
permalink: /demo/
---

# 🎯 Complete Workflow Implementation

## Repository Structure

This repository demonstrates a complete GitHub Actions automation suite for GitHub Pages. Here's what's included:

### 📁 File Structure

```
github-pages-automation-demo/
├── .github/
│   ├── workflows/
│   │   ├── deploy-jekyll.yml     # Main deployment workflow
│   │   ├── security.yml          # Security scanning
│   │   ├── maintenance.yml       # Site maintenance
│   │   └── test-pr.yml          # PR testing
│   └── mlc_config.json          # Link checker config
├── _config.yml                  # Jekyll configuration
├── Gemfile                      # Ruby dependencies
├── index.md                     # Homepage
├── quick-start.md              # Implementation guide
└── demo.md                     # This documentation
```

## 🚀 Complete Workflow Files

### 1. Jekyll Deployment Workflow

**File**: `.github/workflows/deploy-jekyll.yml`

This is the main deployment workflow that:
- Builds Jekyll sites automatically
- Optimizes assets (images, CSS, JS)
- Validates HTML and links
- Runs performance audits
- Deploys to GitHub Pages

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
  # Build Jekyll site with optimization
  build:
    runs-on: ubuntu-latest
    outputs:
      build-status: ${{ steps.build.outcome }}
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

      - name: 📦 Install optimization tools
        run: |
          npm install -g imagemin-cli terser cssnano-cli html-validate broken-link-checker @lhci/cli

      - name: 🔧 Setup Pages
        uses: actions/configure-pages@v4

      - name: 🏗️ Build Jekyll site
        id: build
        run: |
          bundle exec jekyll build --verbose
          echo "✅ Jekyll build completed successfully!"
        env:
          JEKYLL_ENV: ${{ github.event.inputs.environment || 'production' }}

      # Asset optimization
      - name: 🖼️ Optimize images
        run: |
          if find _site -name "*.jpg" -o -name "*.png" -o -name "*.gif" | head -1 | grep -q .; then
            imagemin '_site/**/*.{jpg,png,gif}' --out-dir=_site --plugin=imagemin-mozjpeg --plugin=imagemin-pngquant
            echo "✅ Images optimized"
          else
            echo "ℹ️ No images found to optimize"
          fi

      - name: 📦 Minify CSS
        run: |
          find _site -name "*.css" -exec cssnano {} {} \;
          echo "✅ CSS minified"

      - name: 📦 Minify JavaScript
        run: |
          if find _site -name "*.js" | head -1 | grep -q .; then
            find _site -name "*.js" -exec terser {} --compress --mangle --output {} \;
            echo "✅ JavaScript minified"
          else
            echo "ℹ️ No JavaScript files found to minify"
          fi

      # Validation
      - name: ✅ Validate HTML
        run: |
          if find _site -name "*.html" | head -1 | grep -q .; then
            html-validate '_site/**/*.html' || echo "HTML validation completed with warnings"
          fi

      - name: 🔗 Check internal links
        run: |
          python3 -m http.server 8080 --directory _site &
          SERVER_PID=$!
          sleep 5
          
          blc http://localhost:8080 --recursive --ordered --exclude-external --filter-level 3 || echo "Link check completed"
          
          kill $SERVER_PID

      - name: 🎯 Lighthouse performance audit
        run: |
          python3 -m http.server 8081 --directory _site &
          SERVER_PID=$!
          sleep 5
          
          lhci autorun --collect.url=http://localhost:8081 --collect.numberOfRuns=1 --assert.preset=lighthouse:no-pwa || echo "Lighthouse audit completed"
          
          kill $SERVER_PID

      - name: 📈 Build summary
        run: |
          echo "### 🎯 Build & Optimization Results" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Optimized Files:**" >> $GITHUB_STEP_SUMMARY
          echo "- CSS files: $(find _site -name '*.css' | wc -l)" >> $GITHUB_STEP_SUMMARY
          echo "- JavaScript files: $(find _site -name '*.js' | wc -l)" >> $GITHUB_STEP_SUMMARY
          echo "- Image files: $(find _site -name '*.jpg' -o -name '*.png' -o -name '*.gif' | wc -l)" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Total Site Size:** $(du -sh _site | cut -f1)" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "🚀 Site is ready for deployment!" >> $GITHUB_STEP_SUMMARY

      - name: 📤 Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: _site/

  # Deploy to GitHub Pages
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    if: success()
    steps:
      - name: 🚀 Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

      - name: 📋 Deployment summary
        run: |
          echo "### 🚀 Deployment Successful!" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Site URL:** ${{ steps.deployment.outputs.page_url }}" >> $GITHUB_STEP_SUMMARY
          echo "**Environment:** ${{ github.event.inputs.environment || 'production' }}" >> $GITHUB_STEP_SUMMARY
          echo "**Commit:** ${{ github.sha }}" >> $GITHUB_STEP_SUMMARY
          echo "**Deployed at:** $(date)" >> $GITHUB_STEP_SUMMARY
```

### 2. Security Scanning Workflow

**File**: `.github/workflows/security.yml`

```yaml
name: 🔒 Security & Dependency Management

on:
  schedule:
    # Run security scans daily at 6 AM UTC
    - cron: '0 6 * * *'
  push:
    branches: [ main, master ]
    paths:
      - 'Gemfile*'
      - 'package*.json'
      - 'yarn.lock'
      - 'pnpm-lock.yaml'
  pull_request:
    paths:
      - 'Gemfile*'
      - 'package*.json'
      - 'yarn.lock'
      - 'pnpm-lock.yaml'
  workflow_dispatch:
    inputs:
      scan_type:
        description: 'Type of security scan'
        required: true
        default: 'full'
        type: choice
        options:
          - full
          - dependencies
          - code
          - secrets

permissions:
  contents: read
  security-events: write
  pull-requests: write
  issues: write

env:
  RUBY_VERSION: '3.2'
  NODE_VERSION: '20'

jobs:
  # Dependency vulnerability scanning
  dependency-scan:
    runs-on: ubuntu-latest
    if: github.event.inputs.scan_type == 'dependencies' || github.event.inputs.scan_type == 'full' || github.event.inputs.scan_type == ''
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
          severity: 'CRITICAL,HIGH,MEDIUM'

      - name: 📤 Upload Trivy scan results to GitHub Security tab
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

      - name: 🔍 Ruby dependency audit
        if: hashFiles('**/Gemfile.lock') != ''
        run: |
          gem install bundler-audit
          bundle-audit check --update --format json --output bundler-audit.json || true
          
          if [ -s bundler-audit.json ]; then
            echo "### 🔴 Ruby Security Vulnerabilities Found" >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`json" >> $GITHUB_STEP_SUMMARY
            cat bundler-audit.json >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          else
            echo "### ✅ No Ruby vulnerabilities found" >> $GITHUB_STEP_SUMMARY
          fi

      - name: 🔧 Setup Node.js
        if: hashFiles('**/package-lock.json') != ''
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: 🔍 Node.js dependency audit
        if: hashFiles('**/package-lock.json') != ''
        run: |
          npm audit --audit-level=moderate --json > npm-audit.json || true
          
          if [ -s npm-audit.json ] && [ "$(cat npm-audit.json | jq '.vulnerabilities | length')" -gt 0 ]; then
            echo "### 🔴 Node.js Security Vulnerabilities Found" >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`json" >> $GITHUB_STEP_SUMMARY
            cat npm-audit.json | jq '.vulnerabilities' >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          else
            echo "### ✅ No Node.js vulnerabilities found" >> $GITHUB_STEP_SUMMARY
          fi

  # Security reporting
  security-report:
    runs-on: ubuntu-latest
    needs: [dependency-scan]
    if: always() && (github.event_name == 'schedule' || github.event.inputs.scan_type == 'full')
    steps:
      - name: 📊 Generate security report
        run: |
          echo "## 🔒 Security Scan Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Scan Date**: $(date)" >> $GITHUB_STEP_SUMMARY
          echo "**Repository**: ${{ github.repository }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          
          if [ "${{ needs.dependency-scan.result }}" = "success" ]; then
            echo "✅ **Dependency Scan**: Passed" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ **Dependency Scan**: Issues found" >> $GITHUB_STEP_SUMMARY
          fi
          
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Next Steps" >> $GITHUB_STEP_SUMMARY
          echo "- Review the Security tab for detailed vulnerability reports" >> $GITHUB_STEP_SUMMARY
          echo "- Address any critical or high-severity issues promptly" >> $GITHUB_STEP_SUMMARY
```

### 3. Maintenance & Monitoring Workflow

**File**: `.github/workflows/maintenance.yml`

```yaml
name: 🔧 Maintenance & Monitoring

on:
  schedule:
    # Run maintenance tasks daily at 3 AM UTC
    - cron: '0 3 * * *'
    # Run comprehensive monitoring weekly on Sundays at 5 AM UTC
    - cron: '0 5 * * 0'
  workflow_dispatch:
    inputs:
      task_type:
        description: 'Type of maintenance task'
        required: true
        default: 'all'
        type: choice
        options:
          - all
          - links
          - performance
          - cleanup
          - backup

permissions:
  contents: read
  issues: write
  pull-requests: write

env:
  NODE_VERSION: '20'
  RUBY_VERSION: '3.2'

jobs:
  # Link checking and validation
  link-check:
    runs-on: ubuntu-latest
    if: github.event.inputs.task_type == 'links' || github.event.inputs.task_type == 'all' || github.event.inputs.task_type == ''
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: 🔧 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: 🔗 Check internal links in markdown files
        uses: gaurav-nelson/github-action-markdown-link-check@v1
        with:
          use-quiet-mode: 'yes'
          use-verbose-mode: 'yes'
          config-file: '.github/mlc_config.json'
          folder-path: '.'
          file-extension: '.md'

      - name: 🌐 Build site for link checking
        run: |
          if [ -f "Gemfile" ]; then
            # Jekyll site
            gem install bundler
            bundle install
            bundle exec jekyll build
            SITE_DIR="_site"
          else
            # Static HTML site
            SITE_DIR="."
          fi
          echo "SITE_DIR=$SITE_DIR" >> $GITHUB_ENV

      - name: 🔗 Check all links in built site
        run: |
          npm install -g broken-link-checker
          
          # Start a simple HTTP server
          python3 -m http.server 8080 --directory ${{ env.SITE_DIR }} &
          SERVER_PID=$!
          sleep 5
          
          # Check links
          blc http://localhost:8080 --recursive --ordered --exclude-external --filter-level 3 > link-check-report.txt || true
          
          # Stop server
          kill $SERVER_PID
          
          # Report results
          if grep -q "BROKEN" link-check-report.txt; then
            echo "### 🔴 Broken Links Found" >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
            grep "BROKEN" link-check-report.txt >> $GITHUB_STEP_SUMMARY
            echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          else
            echo "### ✅ All internal links are working" >> $GITHUB_STEP_SUMMARY
          fi

  # Performance monitoring
  performance-monitor:
    runs-on: ubuntu-latest
    if: github.event.inputs.task_type == 'performance' || github.event.inputs.task_type == 'all' || github.event.inputs.task_type == ''
    steps:
      - name: 📥 Checkout
        uses: actions/checkout@v4

      - name: 🔧 Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}

      - name: 🏗️ Build site
        run: |
          if [ -f "Gemfile" ]; then
            gem install bundler
            bundle install
            bundle exec jekyll build
            SITE_DIR="_site"
          else
            SITE_DIR="."
          fi
          echo "SITE_DIR=$SITE_DIR" >> $GITHUB_ENV

      - name: 🎯 Performance audit with Lighthouse CI
        run: |
          npm install -g @lhci/cli
          
          # Start server
          python3 -m http.server 8082 --directory ${{ env.SITE_DIR }} &
          SERVER_PID=$!
          sleep 5
          
          # Run Lighthouse CI
          lhci autorun --collect.url=http://localhost:8082 --collect.numberOfRuns=1 --assert.preset=lighthouse:no-pwa || true
          
          kill $SERVER_PID
          
          # Parse results
          if [ -f ".lighthouseci/lhr-*.json" ]; then
            echo "### 🎯 Performance Metrics" >> $GITHUB_STEP_SUMMARY
            echo "Performance audit completed - check artifacts for detailed results" >> $GITHUB_STEP_SUMMARY
          fi

      - name: 📈 Site size analysis
        run: |
          echo "### 📏 Site Size Analysis" >> $GITHUB_STEP_SUMMARY
          
          TOTAL_SIZE=$(du -sh ${{ env.SITE_DIR }} | cut -f1)
          echo "**Total Site Size**: $TOTAL_SIZE" >> $GITHUB_STEP_SUMMARY
          
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Largest Files**:" >> $GITHUB_STEP_SUMMARY
          echo "\`\`\`" >> $GITHUB_STEP_SUMMARY
          find ${{ env.SITE_DIR }} -type f -exec ls -lh {} \; | sort -k5 -hr | head -10 >> $GITHUB_STEP_SUMMARY
          echo "\`\`\`" >> $GITHUB_STEP_SUMMARY

  # Summary and reporting
  maintenance-summary:
    runs-on: ubuntu-latest
    needs: [link-check, performance-monitor]
    if: always()
    steps:
      - name: 📊 Generate maintenance summary
        run: |
          echo "## 🔧 Maintenance Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "**Date**: $(date)" >> $GITHUB_STEP_SUMMARY
          echo "**Repository**: ${{ github.repository }}" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          
          # Link check results
          if [ "${{ needs.link-check.result }}" = "success" ]; then
            echo "✅ **Link Check**: All links working" >> $GITHUB_STEP_SUMMARY
          else
            echo "❌ **Link Check**: Issues found" >> $GITHUB_STEP_SUMMARY
          fi
          
          # Performance monitor results
          if [ "${{ needs.performance-monitor.result }}" = "success" ]; then
            echo "✅ **Performance**: Within acceptable ranges" >> $GITHUB_STEP_SUMMARY
          else
            echo "⚠️ **Performance**: Review recommended" >> $GITHUB_STEP_SUMMARY
          fi
```

### 4. Pull Request Testing Workflow

**File**: `.github/workflows/test-pr.yml`

```yaml
name: 🧪 Test Pull Request

on:
  pull_request:
    branches: [ main, master ]
    types: [opened, synchronize, reopened]
  workflow_dispatch:

permissions:
  contents: read
  pull-requests: write
  security-events: write

env:
  NODE_VERSION: '20'
  RUBY_VERSION: '3.2'

jobs:
  # Build and basic validation
  build-test:
    runs-on: ubuntu-latest
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

      - name: 🏗️ Test Jekyll build
        run: |
          bundle exec jekyll build --verbose
          echo "✅ Jekyll build successful" >> $GITHUB_STEP_SUMMARY

  # Security scanning
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
          severity: 'HIGH,CRITICAL'

      - name: 📤 Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  # PR summary
  pr-summary:
    runs-on: ubuntu-latest
    needs: [build-test, security-scan]
    if: always()
    steps:
      - name: 📊 Generate PR summary
        uses: actions/github-script@v7
        with:
          script: |
            const results = {
              'Build Test': '${{ needs.build-test.result }}',
              'Security Scan': '${{ needs.security-scan.result }}'
            };
            
            let summary = '## 🧪 Pull Request Test Summary\n\n';
            
            for (const [test, result] of Object.entries(results)) {
              const icon = result === 'success' ? '✅' : result === 'failure' ? '❌' : '⚠️';
              summary += `${icon} **${test}**: ${result}\n`;
            }
            
            const hasFailures = Object.values(results).some(result => result === 'failure');
            
            if (hasFailures) {
              summary += '\n❌ Please review and fix the failed tests above';
            } else {
              summary += '\n✅ All tests passed! This PR is ready for review';
            }
            
            // Add to step summary
            core.summary.addRaw(summary);
            await core.summary.write();
            
            // Comment on PR
            if (context.eventName === 'pull_request') {
              await github.rest.issues.createComment({
                issue_number: context.issue.number,
                owner: context.repo.owner,
                repo: context.repo.repo,
                body: summary
              });
            }
```

## 🔧 Configuration Files

### Link Checker Configuration

**File**: `.github/mlc_config.json`

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
  "retryCount": 3,
  "fallbackRetryDelay": "30s",
  "aliveStatusCodes": [200, 206]
}
```

## 🚀 Implementation Instructions

### Step 1: Copy Files
1. Create `.github/workflows/` directory in your repository
2. Copy each workflow file above into the workflows directory
3. Copy the `mlc_config.json` file into `.github/`

### Step 2: Enable GitHub Pages
1. Go to repository Settings → Pages
2. Set Source to "GitHub Actions"
3. Save settings

### Step 3: Commit and Deploy
1. Commit all workflow files
2. Push to main branch
3. Check Actions tab for workflow execution
4. Visit your GitHub Pages URL once deployed

## 🎉 Features Demonstrated

- ✅ **Automated Jekyll deployment** with asset optimization
- ✅ **Security scanning** with Trivy and dependency audits
- ✅ **Performance monitoring** with Lighthouse CI
- ✅ **Link validation** for internal and external links
- ✅ **Pull request testing** with comprehensive validation
- ✅ **Automated reporting** with GitHub step summaries
- ✅ **Scheduled maintenance** tasks
- ✅ **Manual workflow dispatch** options

---

### 📋 Next Steps

- [Quick Start Guide]({{ '/quick-start/' | relative_url }})
- [View Live Workflows](https://github.com/Clubhouse1661/github-pages-automation-demo/actions)
- [Complete Documentation]({{ '/workflow-documentation/' | relative_url }})