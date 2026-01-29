# QAAtlas Distribution

Pre-built binaries and easy installation for [QAAtlas](https://github.com/mauricecarrier7/qaatlas) - the AI-powered test plan generator.

## Quick Install

### npm (Recommended)

```bash
npm install -g qaatlas
```

### macOS (Universal Binary)

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas/releases/latest/download/qaatlas-macos
chmod +x /usr/local/bin/qaatlas

# Remove quarantine if needed
xattr -d com.apple.quarantine /usr/local/bin/qaatlas 2>/dev/null || true
```

### Linux x64

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas/releases/latest/download/qaatlas-linux-x64
chmod +x /usr/local/bin/qaatlas
```

### Linux ARM64

```bash
curl -L -o /usr/local/bin/qaatlas \
  https://github.com/mauricecarrier7/qaatlas/releases/latest/download/qaatlas-linux-arm64
chmod +x /usr/local/bin/qaatlas
```

### Windows

Download `qaatlas-win-x64.exe` from the [latest release](https://github.com/mauricecarrier7/qaatlas/releases/latest) and add to your PATH.

## Verify Installation

```bash
qaatlas --version
```

## Quick Start

1. Set your Anthropic API key:
   ```bash
   export ANTHROPIC_API_KEY=your-api-key
   ```

2. Analyze your code changes:
   ```bash
   # From a git repository
   qaatlas analyze

   # Or with a diff file
   qaatlas analyze --diff changes.patch
   ```

3. Check outputs in `./qaatlas-output/`

## CI/CD Integration

### GitHub Actions

```yaml
- name: Install QAAtlas
  run: npm install -g qaatlas@1.0.0

- name: Run Analysis
  env:
    ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
  run: |
    git diff origin/${{ github.base_ref }}...HEAD > diff.patch
    qaatlas analyze --diff diff.patch
```

### With Caching

```yaml
- name: Cache QAAtlas
  id: cache
  uses: actions/cache@v4
  with:
    path: /usr/local/bin/qaatlas
    key: qaatlas-1.0.0-${{ runner.os }}-${{ runner.arch }}

- name: Install QAAtlas
  if: steps.cache.outputs.cache-hit != 'true'
  run: |
    curl -L -o /usr/local/bin/qaatlas \
      https://github.com/mauricecarrier7/qaatlas/releases/download/v1.0.0/qaatlas-${{ runner.os == 'macOS' && 'macos' || 'linux-x64' }}
    chmod +x /usr/local/bin/qaatlas
```

## Documentation

For full documentation, configuration options, and API reference, see the [main repository](https://github.com/mauricecarrier7/qaatlas).

## License

MIT
