# QAAtlas

AI-powered test coverage analyzer for Swift and Objective-C iOS projects. Generates test plans, risk assessments, coverage maps, and identifies test gaps.

## Features

- **Swift & Objective-C Support**: Full support for iOS, macOS, watchOS, and tvOS projects
- **Static Gap Analysis**: Identifies symbols lacking test coverage without running code
- **Semantic Test Coverage** (v2.0+): 3-tier analysis that tracks actual object instantiations and method calls
- **Test Plan Generation**: AI-powered test case recommendations based on code changes
- **Risk Assessment**: Identifies potential risks and provides mitigation strategies
- **Coverage Mapping**: Maps changed files to recommended tests and identifies coverage gaps
- **LLM Response Caching** (v2.2+): Cache analysis results to save costs and ensure consistency
- **Built-in Feedback** (v2.1+): Report issues and request features directly from CLI
- **Baseline Comparison**: Track coverage improvements over time with delta reporting
- **Multiple Output Formats**: JSON, Markdown, PR comments, and coverage maps
- **CI/CD Integration**: Works seamlessly with GitHub Actions and other CI systems
- **Flexible Configuration**: YAML-based configuration with iOS presets

## Supported Languages

| Language | Gap Analysis | Semantic Coverage | Test Detection |
|----------|:------------:|:-----------------:|:--------------:|
| **Swift** | ✅ | ✅ (Tier 1-3) | ✅ |
| **Objective-C** | ✅ | ✅ (Tier 1-3) | ✅ |
| TypeScript/JavaScript | — | — | ✅ |
| Kotlin/Java | — | — | ✅ |
| Python | — | — | ✅ |
| Go | — | — | ✅ |
| Rust | — | — | ✅ |

## What's New in v2.2

### LLM Response Caching

Cache analysis results to avoid redundant API calls:

```bash
qaatlas analyze --diff changes.patch --cache
qaatlas analyze --clear-cache  # Clear all cached results
```

### Built-in Feedback System

Report issues directly from the CLI:

```bash
qaatlas feedback --type accuracy --title "Detection issue"
qaatlas feedback --type bug --title "Something broke"
qaatlas feedback --type feature --title "Add feature X"
```

### Semantic Test Coverage Detection (v2.0)

QAAtlas v2.0 introduced a revolutionary 3-tier semantic test coverage system:

| Tier | Detection Method | Confidence |
|------|-----------------|------------|
| **Tier 3** | Dependency graph - tracks transitive coverage | High |
| **Tier 2** | Body analysis - finds actual instantiations & calls | High |
| **Tier 1** | Name matching - file/method name patterns | Medium |

**Example:** If your test does this:

```swift
func testUserService() {
    let service = UserService()       // Tier 2: UserService detected
    let user = service.fetchUser()    // Tier 2: fetchUser() detected
    XCTAssertEqual(user.name, "John") // Tier 2: User.name detected
}
```

QAAtlas will correctly identify `UserService`, `fetchUser`, and `User` as covered - even if the test file isn't named `UserServiceTests.swift`.

### Supports Swift and Objective-C

```objc
// Objective-C patterns detected:
[[UserService alloc] init]
[service fetchUserWithId:@"123"]
```

### Mock Filtering

Automatically excludes test doubles:
- `MockUserService`, `UserServiceMock`, `UserServiceStub`, etc.

## Installation

### npm (Recommended)

```bash
npm install -g qaatlas
```

Or use directly with npx:

```bash
npx qaatlas analyze
```

### Direct Binary Download

Pre-built binaries are available for all major platforms. No Node.js required.

**macOS (Universal - Intel & Apple Silicon)**
```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/latest/download/qaatlas-macos
chmod +x /usr/local/bin/qaatlas

# Remove quarantine if needed
xattr -d com.apple.quarantine /usr/local/bin/qaatlas 2>/dev/null || true
```

**Linux x64**
```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/latest/download/qaatlas-linux-x64
chmod +x /usr/local/bin/qaatlas
```

**Linux ARM64**
```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas-dist/releases/latest/download/qaatlas-linux-arm64
chmod +x /usr/local/bin/qaatlas
```

**Windows x64**

Download `qaatlas-win-x64.exe` from the [latest release](https://github.com/mauricecarrier7/qaatlas-dist/releases/latest) and add to your PATH.

### Build from Source

```bash
git clone https://github.com/mauricecarrier7/qaatlas.git
cd qaatlas
npm install
npm run build
npm link  # or: node dist/cli/index.js
```

## Verify Installation

```bash
qaatlas --version
qaatlas --help

# Health check (validates SEA binary startup)
qaatlas --health
```

## Quick Start

1. Set your API key:
   ```bash
   # For Claude (default)
   export ANTHROPIC_API_KEY=your-api-key

   # Or for OpenAI
   export OPENAI_API_KEY=your-api-key
   ```

2. Run analysis on your changes:
   ```bash
   # Analyze staged/unstaged git changes
   qaatlas analyze

   # Analyze a specific diff file
   qaatlas analyze --diff changes.patch

   # Pipe diff from stdin
   git diff main...HEAD | qaatlas analyze

   # Use OpenAI instead of Claude
   qaatlas analyze --provider openai --model gpt-4o
   ```

3. Check the output in `./qaatlas-output/`:
   - `report.json` - Full structured report
   - `testplan.md` - Human-readable test plan
   - `risk.md` - Risk assessment document
   - `pr-comment.md` - Condensed PR comment
   - `coverage-map.json` - File-to-test mapping

### For iOS Projects

```bash
# Run gap analysis on your iOS project
qaatlas gap-analysis --target ./Sources --preset ios

# Check coverage with baseline comparison
qaatlas gap-analysis --target ./Sources --baseline ./previous.json
```

## Gap Analysis (iOS/Swift/Objective-C)

Static test gap analysis for iOS projects. Identifies symbols (classes, structs, enums, protocols, actors, functions) lacking test coverage by comparing production code against test files.

**Supported:**
- Swift 5.x (classes, structs, enums, protocols, actors, extensions)
- Objective-C (classes, protocols, categories)
- Mixed Swift/Objective-C projects

### Basic Usage

```bash
# Analyze a Swift project
qaatlas gap-analysis --target ./MyApp

# Use iOS preset (excludes Pods, Carthage, DerivedData, generated code)
qaatlas gap-analysis --target ./MyApp --preset ios

# Analyze with baseline comparison (shows delta)
qaatlas gap-analysis --target ./MyApp --baseline ./previous-report.json

# Filter by priority level
qaatlas gap-analysis --target ./MyApp --min-priority medium

# Show only top 10 gaps
qaatlas gap-analysis --target ./MyApp --top 10
```

### Options

```
qaatlas gap-analysis [options]

Options:
  -t, --target <path>        Target directory to analyze (required)
  --preset <name>            Use preset configuration (ios)
  -f, --formats <formats>    Output formats: json,markdown (default: "json,markdown")
  -o, --output-dir <path>    Output directory (default: "./qaatlas-output")
  --module <name>            Filter to specific module
  --min-priority <level>     Minimum priority: high,medium,low (default: "low")
  --baseline <path>          Compare against baseline JSON for delta
  --include-extensions       Include type extensions in analysis
  -v, --verbose              Enable verbose logging
  -h, --help                 Display help
```

### Priority Levels

Gaps are automatically prioritized based on code location:

| Priority | Targets |
|----------|---------|
| **High** | Domain models, Services, business logic |
| **Medium** | Data layer, Repositories, Networking |
| **Low** | UI/Views, ViewModels, SwiftUI views |

### Exit Codes

- `0` - **PASS**: No high or medium priority gaps
- `2` - **FAIL**: High priority gaps found
- `3` - **WARN**: Medium priority gaps (no high)

Use exit codes in CI to enforce coverage policies:

```bash
qaatlas gap-analysis --target ./MyApp --min-priority medium || exit 1
```

### Example Output

The Markdown report includes:

```markdown
# Test Gap Analysis Report

## Summary

| Metric | Value |
|--------|-------|
| Total Symbols | 45 |
| Tested | 32 |
| Untested | 13 |
| Coverage | 71.1% |

## High Priority Gaps

### Domain/Services
- `OrderService` - 40% covered (missing: processRefund, validateOrder)
- `PaymentValidator` - 0% covered

## Recommendations

1. Add tests for OrderService.processRefund (high priority, core business logic)
2. Create PaymentValidatorTests.swift for payment validation coverage
```

### CI Integration

```yaml
- name: Check Test Coverage Gaps
  run: |
    qaatlas gap-analysis \
      --target ./Sources \
      --preset ios \
      --min-priority medium
  # Fails if high-priority gaps exist
```

## Configuration

Create a `qaatlas.yml` file in your project root:

```yaml
version: "1"

provider:
  name: claude
  model: claude-sonnet-4-20250514
  maxTokens: 4096

analysis:
  maxFiles: 20
  maxLinesPerFile: 500
  includePatterns:
    - "**/*.ts"
    - "**/*.tsx"
    - "**/*.js"
  excludePatterns:
    - "**/node_modules/**"
    - "**/*.test.*"

output:
  dir: ./qaatlas-output
  formats:
    - report
    - testplan
    - risk
    - pr-comment
    - coverage-map

context:
  includeImports: true
  includeFunctionSignatures: true
  surroundingLines: 10

# Optional: Customize exit codes for CI
exitCodes:
  success: 0
  error: 1
  criticalRisk: 2           # Critical risk in analyze
  highPriorityGaps: 2       # High priority gaps in gap-analysis
  mediumPriorityGaps: 3     # Medium priority (warning)
  missingCriticalTests: 2   # Missing critical tests
  belowCoverageThreshold: 2 # Coverage below --min-coverage
```

## CLI Options

```
qaatlas [options] [command]

Global Options:
  -V, --version            Output version number
  --health                 Quick startup health check (exit 0 if healthy)
  -h, --help               Display help

qaatlas analyze [options]

Options:
  -c, --config <path>      Path to config file (qaatlas.yml)
  -d, --diff <path>        Path to diff file
  -o, --output-dir <path>  Output directory for generated files
  -p, --provider <name>    LLM provider (default: claude)
  -m, --model <name>       Model to use for analysis
  -f, --formats <formats>  Comma-separated output formats
  -v, --verbose            Enable verbose logging
  -q, --quiet              Quiet mode - suppress logs, output only JSON
  --dry-run                Parse diff without calling LLM
  --baseline <path>        Compare against previous report.json
  --feedback               Prompt to submit feedback after analysis
  --cache                  Enable caching of LLM results
  --cache-dir <path>       Directory for cache files (default: .qaatlas-cache)
  --no-cache               Force fresh LLM call (bypass cache)
  --clear-cache            Clear all cached results and exit
  -h, --help               Display help

qaatlas gap-analysis [options]

Options:
  -t, --target <path>         Target directory to analyze (required)
  --test-dir <path>           Custom test directory
  --preset <name>             Use preset configuration (ios, swift-package)
  -f, --formats <formats>     Output formats: json,markdown (default)
  -o, --output-dir <path>     Output directory (default: ./qaatlas-output)
  --module <name>             Filter to specific module
  --min-priority <level>      Minimum priority: high, medium, low
  --min-coverage <percent>    Minimum coverage percentage threshold
  --fail-below                Fail if coverage is below --min-coverage
  --top <n>                   Show only top N gaps in summary (default: 20)
  --public-only               Only analyze public/open symbols
  --min-access-level <level>  Minimum: open, public, internal, fileprivate, private
  --baseline <path>           Compare against baseline JSON file
  --feedback                  Prompt to submit feedback after analysis
  -v, --verbose               Enable verbose logging
  -q, --quiet                 Quiet mode - suppress logs, output only JSON
  -h, --help                  Display help

qaatlas feedback [options]

Options:
  -t, --type <type>          Feedback type: accuracy, bug, or feature
  --title <title>            Issue title
  --description <desc>       Issue description
  --dry-run                  Preview without creating issue
  --include-version          Include QAAtlas version in report
  -h, --help                 Display help
```

## Submit Feedback (v2.1)

QAAtlas includes a built-in feedback system to help improve detection accuracy:

```bash
# Interactive feedback
qaatlas feedback

# Report an accuracy issue
qaatlas feedback --type accuracy --title "False positive in CarPlay detection"

# Report a bug
qaatlas feedback --type bug --title "Crash on large diffs"

# Request a feature
qaatlas feedback --type feature --title "Add Python support"

# Preview without submitting
qaatlas feedback --dry-run
```

**Integrated feedback after analysis:**

```bash
# Prompt for feedback after gap-analysis
qaatlas gap-analysis --target ./src --feedback

# Prompt for feedback after analyze
qaatlas analyze --diff changes.patch --feedback
```

**Requirements:**
- GitHub CLI (`gh`) must be installed and authenticated
- Issues are created on the QAAtlas repository

## LLM Response Caching (v2.2)

Cache LLM analysis results to avoid redundant API calls:

```bash
# Enable caching for repeated analyses
qaatlas analyze --diff changes.patch --cache

# Use custom cache directory
qaatlas analyze --cache --cache-dir ./my-cache

# Force fresh analysis (bypass cache)
qaatlas analyze --cache --no-cache

# Clear all cached results
qaatlas analyze --clear-cache
```

**Benefits:**
- **Cost savings**: Avoid duplicate LLM API calls
- **Consistency**: Same input = same output (deterministic)
- **Speed**: Instant results for unchanged diffs

**Cache Key Includes:**
- Diff content hash
- Model name
- QAAtlas version (auto-invalidates on upgrade)
- Config options (maxFiles, maxLinesPerFile, preprocess)

**Cache Behavior:**
- Stored in `.qaatlas-cache/` (configurable)
- 7-day TTL by default
- JSON format with metadata for debugging

## Tracking Coverage Improvements

QAAtlas can track test coverage improvements across PR updates using baseline comparisons.

### First Run

```bash
qaatlas analyze --diff changes.patch -o ./first-run
# Output: Missing: 5 tests
```

### After Adding Tests

```bash
qaatlas analyze --diff changes.patch --baseline ./first-run/report.json
# Output:
# 📈 Coverage: 3 → 7 tests covered (+4)
#    Risk Level: high → medium
# ✅ Test coverage improved since last run!
```

### PR Comment Output

The PR comment includes coverage metrics:

```markdown
### Test Coverage Status
✅ **PASS** - 5 of 5 test recommendations covered

| Status | Count |
|:-------|------:|
| Already covered | 5 |
| Partially covered | 0 |
| Missing | 0 |

### Baseline Comparison
📈 **Coverage:** 3 → 5 tests covered (+2)
- Risk level: HIGH → MEDIUM

🎉 *Test coverage improved since last run!*
```

## GitHub Actions Integration

### Basic Setup

```yaml
name: QAAtlas Test Plan

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install -g qaatlas

      - run: |
          git diff origin/${{ github.base_ref }}...HEAD > diff.patch
          qaatlas analyze --diff diff.patch
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}

      - uses: actions/upload-artifact@v4
        with:
          name: qa-reports
          path: ./qaatlas-output

      - name: Comment PR
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const comment = fs.readFileSync('./qaatlas-output/pr-comment.md', 'utf8');
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: comment
            });
```

### With Binary Caching (Faster)

```yaml
- name: Cache QAAtlas
  id: cache
  uses: actions/cache@v4
  with:
    path: /usr/local/bin/qaatlas
    key: qaatlas-2.2.1-${{ runner.os }}-${{ runner.arch }}

- name: Install QAAtlas
  if: steps.cache.outputs.cache-hit != 'true'
  run: |
    curl -L -o /usr/local/bin/qaatlas \
      https://github.com/mauricecarrier7/qaatlas-dist/releases/latest/download/qaatlas-linux-x64
    chmod +x /usr/local/bin/qaatlas

- name: Run QAAtlas
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: |
    git diff origin/${{ github.base_ref }}...HEAD > diff.patch
    qaatlas analyze --diff diff.patch
```

### iOS Project Gap Analysis

```yaml
name: iOS Test Coverage Check

on:
  pull_request:
    paths:
      - '**/*.swift'
      - '**/*.m'
      - '**/*.h'

jobs:
  coverage:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install QAAtlas
        run: npm install -g qaatlas

      - name: Check Test Coverage Gaps
        run: |
          qaatlas gap-analysis \
            --target ./Sources \
            --preset ios \
            --min-priority medium \
            --baseline ./previous-coverage.json || true

      - name: Upload Coverage Report
        uses: actions/upload-artifact@v4
        with:
          name: coverage-report
          path: ./qaatlas-output/
```

See `examples/workflows/qaatlas.yml` for a complete workflow example.

## Output Formats

### report.json
Complete structured report with all test cases, risks, and coverage data. Use this for programmatic access or further processing.

### testplan.md
Human-readable test plan with:
- Test cases organized by priority
- Step-by-step test procedures
- Assertion checklists
- Quick checklist summary

### risk.md
Risk assessment document with:
- Risk summary table
- Detailed risk descriptions
- Mitigation strategies
- Risk statistics

### pr-comment.md
Condensed GitHub PR comment with:
- Risk level badge
- Important risks (critical/high)
- Priority test cases
- Collapsible details for full information

### coverage-map.json
File-to-test mapping showing:
- Which tests cover each file
- Coverage status (full/partial/none/unknown)
- Uncovered areas

## Exit Codes

- `0` - Success
- `1` - Error (config, parsing, API)
- `2` - Critical risk level detected

## Environment Variables

- `ANTHROPIC_API_KEY` - Your Anthropic API key (for Claude provider)
- `OPENAI_API_KEY` - Your OpenAI API key (for OpenAI provider)
- `NODE_ENV` - Set to `production` for JSON logging

**Note:** Only one API key is required depending on which provider you use.

## Troubleshooting

### macOS Gatekeeper

If you see "cannot be opened because the developer cannot be verified":

```bash
xattr -d com.apple.quarantine /usr/local/bin/qaatlas
```

Or right-click the binary in Finder and select "Open".

### Permission Denied

```bash
chmod +x /usr/local/bin/qaatlas
```

### CI Cache Contains Bad Binary

Bust the cache by changing the cache key:

```yaml
key: qaatlas-1.0.0-${{ runner.os }}-${{ runner.arch }}-v2
```

### Binary Hangs on Startup

If the SEA binary hangs during startup (stuck on Unix domain sockets or IPC initialization):

1. **Use health check to diagnose:**
   ```bash
   qaatlas --health
   # Should output "QAAtlas health check: OK" within 5 seconds
   ```

2. **Startup timeout protection:**
   The binary automatically exits after 5 seconds if initialization hangs. You'll see:
   ```
   QAAtlas: Startup timeout after 5 seconds - exiting
   ```

3. **Fallback to npm:**
   If the binary consistently hangs, use npm instead:
   ```bash
   npx qaatlas analyze
   # or
   npm install -g qaatlas && qaatlas analyze
   ```

4. **CI health check:**
   Add a health check step before running analysis:
   ```yaml
   - name: Verify QAAtlas
     run: qaatlas --health
     timeout-minutes: 1
   ```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Run tests (`npm test`)
4. Commit your changes (`git commit -m 'Add amazing feature'`)
5. Push to the branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

## Release Process

Releases are automated via GitHub Actions. To create a new release:

1. Update version: `npm version patch|minor|major`
2. Push the tag: `git push --tags`
3. The release workflow will:
   - Run tests
   - Build binaries for all platforms
   - Create a GitHub release
   - Publish to npm

Or trigger manually via workflow_dispatch.

## License

MIT
