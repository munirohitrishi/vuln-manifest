# Vulnerable Dependency Lab - SCA Practice
## For Interview Preparation: How to Install, Scan, and Fix Manifest Files

> **WARNING**: This project contains INTENTIONALLY VULNERABLE dependencies.
> **DO NOT** deploy to production. Use only for learning and practice.

---

## Prerequisites: Install Snyk CLI

```bash
# Option 1: npm (recommended)
npm install -g snyk

# Option 2: Standalone (no Node.js needed)
curl -sL https://static.snyk.io/cli/latest/snyk-linux -o snyk
chmod +x snyk
sudo mv snyk /usr/local/bin/

# Authenticate with Snyk (free account at snyk.io)
snyk auth
# This opens a browser - log in and authorize the CLI
```

---

## Lab 1: Java Maven (pom.xml)

```bash
cd java-maven/

# STEP 1: View the manifest file
cat pom.xml
# Notice: log4j-core 2.14.1, jackson-databind 2.13.0, commons-text 1.9

# STEP 2: Install dependencies (requires Maven installed)
mvn dependency:resolve

# STEP 3: View the full dependency tree
mvn dependency:tree
# This shows ALL dependencies including transitive ones
# Look for vulnerable versions deep in the tree

# STEP 4: Filter for a specific library
mvn dependency:tree -Dincludes=org.apache.logging.log4j
# Shows exactly how log4j enters your project

# STEP 5: Scan with Snyk
snyk test --file=pom.xml
# Output shows: CVE IDs, severity, affected package, fix version

# STEP 6: Get detailed report
snyk test --file=pom.xml --json > snyk-report.json

# STEP 7: Monitor continuously
snyk monitor --file=pom.xml
# Uploads to Snyk dashboard for continuous monitoring

# === HOW TO FIX ===

# FIX 1: Direct dependency - change version in pom.xml
# Change: <version>2.14.1</version>
# To:     <version>2.21.1</version>  (for log4j-core)

# FIX 2: Transitive dependency - add to dependencyManagement
# Add this section to pom.xml:
# <dependencyManagement>
#   <dependencies>
#     <dependency>
#       <groupId>com.fasterxml.jackson.core</groupId>
#       <artifactId>jackson-databind</artifactId>
#       <version>2.16.1</version>
#     </dependency>
#   </dependencies>
# </dependencyManagement>

# FIX 3: Exclude + replace
# Add <exclusions> to the parent dependency, then add safe version directly

# STEP 8: Verify fix
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core
snyk test --file=pom.xml
# Should show reduced vulnerabilities
```

---

## Lab 2: JavaScript npm (package.json)

```bash
cd node-npm/

# STEP 1: View the manifest
cat package.json
# Notice: lodash 4.17.20, axios 0.21.1, express 4.17.1

# STEP 2: Install dependencies
npm install
# Creates node_modules/ and package-lock.json

# STEP 3: Built-in vulnerability check
npm audit
# Shows: severity, package, dependency path, fix available

# STEP 4: Scan with Snyk
snyk test
# More detailed than npm audit - shows CVSS, exploit maturity

# STEP 5: View dependency tree for specific package
npm ls lodash
# Shows which packages depend on lodash and at what version

# STEP 6: Auto-fix (safe upgrades within semver range)
npm audit fix
# Automatically upgrades to nearest safe version

# STEP 7: Force fix (may include breaking changes)
npm audit fix --force
# Allows major version upgrades - TEST AFTER THIS!

# === MANUAL FIX FOR TRANSITIVE DEPS ===

# Add "overrides" to package.json:
# "overrides": {
#   "minimist": "1.2.8",
#   "qs": "6.11.0"
# }
# Then: npm install

# STEP 8: Verify
npm audit
# Should show 0 vulnerabilities (or fewer)
npm ls lodash
# Should show patched version only
snyk test
# Final verification
```

---

## Lab 3: Python pip (requirements.txt)

```bash
cd python-pip/

# STEP 1: View the manifest
cat requirements.txt
# Notice: Django==3.2.0, Flask==2.0.0, PyYAML==5.3

# STEP 2: Create virtual environment (ALWAYS do this!)
python3 -m venv .venv
source .venv/bin/activate   # Linux/Mac
# .venv\Scripts\activate    # Windows

# STEP 3: Install dependencies
pip install -r requirements.txt

# STEP 4: See what's installed (with exact versions)
pip freeze

# STEP 5: Scan with pip-audit
pip install pip-audit
pip-audit
# Shows: package, installed version, fix version, CVE IDs

# STEP 6: Scan with Snyk
snyk test --file=requirements.txt
# Detailed vulnerability report

# STEP 7: Scan with Safety (another option)
pip install safety
safety check -r requirements.txt

# === HOW TO FIX ===

# FIX: Update version in requirements.txt
# Change: Django==3.2.0
# To:     Django==4.2.11
# Change: PyYAML==5.3
# To:     PyYAML==6.0.1

# Then reinstall:
pip install -r requirements.txt

# STEP 8: Verify fix
pip-audit
snyk test --file=requirements.txt

# STEP 9: Generate fixed requirements with all exact versions
pip freeze > requirements-fixed.txt

# Deactivate virtual environment when done
deactivate
```

---

## Lab 4: Python Poetry (pyproject.toml)

```bash
cd python-poetry/

# STEP 1: Install Poetry (if not installed)
pip install poetry

# STEP 2: View the manifest
cat pyproject.toml

# STEP 3: Install dependencies
poetry install
# Creates poetry.lock with exact versions + hashes

# STEP 4: View dependency tree
poetry show --tree

# STEP 5: Scan with Snyk
snyk test --file=pyproject.toml

# === HOW TO FIX ===

# FIX: Update specific package
poetry update Django
# Or update all:
poetry update

# Verify:
poetry show django    # Check version
snyk test --file=pyproject.toml
```

---

## Lab 5: Go (go.mod)

```bash
cd go-app/

# STEP 1: View the manifest
cat go.mod
# Notice: gin v1.7.0, jwt-go v3.2.0

# STEP 2: Download dependencies
go mod download

# STEP 3: View dependency graph
go mod graph
# Shows all dependencies and their relationships

# STEP 4: Scan with govulncheck (Go's official scanner)
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...
# UNIQUE: Shows only vulnerabilities in code paths you ACTUALLY CALL

# STEP 5: Scan with Snyk
snyk test --file=go.mod

# === HOW TO FIX ===

# FIX 1: Upgrade specific dependency
go get github.com/gin-gonic/gin@v1.9.1

# FIX 2: Replace vulnerable dependency (in go.mod)
# Add: replace github.com/dgrijalva/jwt-go => github.com/golang-jwt/jwt/v4 v4.5.0

# FIX 3: Exclude a vulnerable version
# Add: exclude gopkg.in/yaml.v2 v2.2.2

# Clean up
go mod tidy

# Verify
govulncheck ./...
snyk test --file=go.mod
```

---

## Lab 6: .NET NuGet (.csproj)

```bash
cd dotnet-app/

# STEP 1: View the manifest
cat VulnApp.csproj
# Notice: Newtonsoft.Json 12.0.3, System.Net.Http 4.3.0

# STEP 2: Install dependencies
dotnet restore

# STEP 3: Built-in vulnerability check (EXCELLENT feature!)
dotnet list package --vulnerable
# Shows vulnerable packages with advisory links

# STEP 4: Include transitive dependencies
dotnet list package --vulnerable --include-transitive

# STEP 5: Scan with Snyk
snyk test --file=VulnApp.csproj

# === HOW TO FIX ===

# FIX: Upgrade package
dotnet add package Newtonsoft.Json --version 13.0.3
dotnet add package System.Net.Http --version 4.3.4

# Verify
dotnet list package --vulnerable
snyk test --file=VulnApp.csproj
```

---

## Lab 7: Docker (Dockerfile)

```bash
cd docker-app/

# STEP 1: View the Dockerfile
cat Dockerfile
# Notice: ubuntu:20.04, running as root, Node 14

# STEP 2: Build the image
docker build -t vuln-app .

# STEP 3: Scan with Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image vuln-app

# STEP 4: Scan with Snyk
snyk container test vuln-app

# STEP 5: Generate SBOM
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  anchore/syft vuln-app -o cyclonedx-json > sbom.json

# === HOW TO FIX ===
# 1. Use specific version + digest: FROM ubuntu:22.04@sha256:...
# 2. Add USER directive: USER 1001
# 3. Use multi-stage build to minimize image size
# 4. Don't COPY everything - use .dockerignore
# 5. Use npm ci instead of npm install
# 6. Remove debug port (9229)
```

---

## Quick Reference: Snyk Commands

```bash
# Authenticate
snyk auth

# Test specific file
snyk test --file=pom.xml
snyk test --file=package.json
snyk test --file=requirements.txt
snyk test --file=go.mod
snyk test --file=VulnApp.csproj

# Test with severity filter
snyk test --severity-threshold=high

# JSON output (for CI/CD)
snyk test --json > report.json

# Monitor (upload to dashboard)
snyk monitor

# Container scanning
snyk container test image-name

# Infrastructure as Code
snyk iac test Dockerfile

# Fix suggestions
snyk fix  # (for npm projects)
```
