# GitHub Dependency Version Checker

A GitHub Action that automatically checks for outdated dependencies in your projects and reports them in pull requests and issues.

## ✨ Features

- 🔍 Automatically detects outdated dependencies
- 📦 Supports multiple package managers:
  - NPM (package.json)
  - Python (requirements.txt)
- 💬 Creates detailed reports as:
  - Pull request comments
  - Issues (for direct pushes to main)
- ⚡ Runs automatically on PRs and pushes
- 🎯 Clear visual indicators for outdated packages

## 📥 Installation

To add this dependency checker to your repository:

1. Create a new file in your repository at `.github/workflows/dependency-check.yml`
2. Copy and paste this content:

```yaml
name: Dependency Version Checker

on:
  pull_request:
    types: [opened, synchronize]
  workflow_dispatch:
  push:
    branches:
      - main
      - master

jobs:
  check-dependencies:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
      issues: write
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.x'
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install requests packaging

      - name: Create and run checker
        run: |
          mkdir -p .github/scripts
          echo '
          import os, json, subprocess, re
          from pathlib import Path

          def check_deps():
              results = {"npm": [], "pip": []}
              
              # Check NPM
              if os.path.exists("package.json"):
                  with open("package.json") as f:
                      pkg = json.load(f)
                      deps = {**pkg.get("dependencies", {}), **pkg.get("devDependencies", {})}
                      for name, ver in deps.items():
                          ver = ver.replace("^", "").replace("~", "")
                          try:
                              proc = subprocess.run(["npm", "view", name, "version"], 
                                                  capture_output=True, text=True)
                              latest = proc.stdout.strip()
                              results["npm"].append({
                                  "name": name,
                                  "current": ver,
                                  "latest": latest
                              })
                          except: pass

              # Check Python
              if os.path.exists("requirements.txt"):
                  with open("requirements.txt") as f:
                      for line in f:
                          if line.strip() and not line.startswith("#"):
                              match = re.match(r"([^=<>]+)(==|>=|<=|~=|!=|<|>)?(.+)?", line)
                              if match:
                                  name = match.group(1).strip()
                                  ver = match.group(3).strip() if match.group(3) else "Not specified"
                                  try:
                                      proc = subprocess.run(["pip", "index", "versions", name],
                                                          capture_output=True, text=True)
                                      versions = re.findall(r"\d+\.\d+\.\d+", proc.stdout)
                                      if versions:
                                          latest = sorted(versions)[-1]
                                          results["pip"].append({
                                              "name": name,
                                              "current": ver,
                                              "latest": latest
                                          })
                                  except: pass
              
              with open("dep-report.json", "w") as f:
                  json.dump(results, f)

          check_deps()
          ' > check_deps.py
          
          python check_deps.py

      - name: Post Results
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            let comment = '# 📦 Dependency Version Check Report\n\n';
            
            try {
              const results = JSON.parse(fs.readFileSync('dep-report.json', 'utf8'));
              
              for (const [manager, packages] of Object.entries(results)) {
                if (packages.length > 0) {
                  comment += `### ${manager.toUpperCase()} Dependencies\n`;
                  comment += '| Package | Current Version | Latest Version | Status |\n';
                  comment += '|---------|----------------|----------------|--------|\n';
                  
                  for (const pkg of packages) {
                    const status = pkg.current !== pkg.latest ? '⚠️ Update Available' : '✅ Up to date';
                    comment += `| ${pkg.name} | ${pkg.current} | ${pkg.latest} | ${status} |\n`;
                  }
                  comment += '\n';
                }
              }
            } catch (error) {
              comment += '⚠️ No dependency files found or error checking dependencies.\n\n';
              comment += 'Make sure you have one of these files:\n';
              comment += '- package.json (for NPM packages)\n';
              comment += '- requirements.txt (for Python packages)\n';
            }
            
            if (context.payload.pull_request) {
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: context.payload.pull_request.number,
                body: comment
              });
            } else {
              await github.rest.issues.create({
                owner: context.repo.owner,
                repo: context.repo.repo,
                title: '📦 Dependency Version Check Report',
                body: comment
              });
            }
```

3. Enable GitHub Actions:
   - Go to your repository's Settings
   - Click Actions → General
   - Select "Allow all actions and reusable workflows"
   - Under "Workflow permissions":
     - Select "Read and write permissions"
     - Check "Allow GitHub Actions to create and approve pull requests"
   - Click "Save"

## 🚀 Usage

The action will run automatically on:
- Every new pull request
- Updates to existing pull requests
- Pushes to main/master branch
- Manual trigger from Actions tab

### Manual Trigger:
1. Go to Actions tab
2. Click "Dependency Version Checker"
3. Click "Run workflow"
4. Select branch
5. Click "Run workflow"

## 📋 Output Example

The action creates reports like this:

### NPM Dependencies
| Package | Current Version | Latest Version | Status |
|---------|----------------|----------------|--------|
| react | 17.0.2 | 18.2.0 | ⚠️ Update Available |
| express | 4.17.1 | 4.18.2 | ⚠️ Update Available |

### Python Dependencies
| Package | Current Version | Latest Version | Status |
|---------|----------------|----------------|--------|
| requests | 2.26.0 | 2.32.3 | ⚠️ Update Available |
| pytest | 7.1.1 | 8.4.0 | ⚠️ Update Available |

## 🔧 Requirements

Your repository should have one or more of:
- `package.json` for NPM packages
- `requirements.txt` for Python packages

## 📝 License

MIT License - feel free to use in any project!

## 🤝 Contributing

Contributions welcome! Feel free to:
1. Fork the repository
2. Create a feature branch
3. Submit a Pull Request

## ⚠️ Troubleshooting

If the action isn't working:
1. Check Actions tab for error messages
2. Verify workflow permissions are correct
3. Ensure dependency files exist
4. Check file paths and syntax

## 📫 Support

Open an issue if you:
- Find a bug
- Want to request a feature
- Need help with setup
sdkjhk


jjksdjhkshkdhvj sj j
ahskjhssdlkjkl
askhkjah
ashjhsjhjh
ashjhasjjahgsjh

askhh
ldjlsjk
sdkhkhs
dfkjhkdh
