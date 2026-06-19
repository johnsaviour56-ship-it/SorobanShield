# SorobanShield - Quick Start Guide

Get up and running with SorobanShield in 5 minutes.

## Installation

### Option 1: From Crates.io (Recommended)
```bash
cargo install sorobanshield
```

### Option 2: From Source
```bash
git clone https://github.com/johnsaviour56-ship-it/sorobanshield.git
cd sorobanshield
cargo build --release
./target/release/sorobanshield --help
```

### Option 3: Development Build
```bash
git clone https://github.com/stellar/sorobanshield.git
cd sorobanshield
cargo build
cargo run -- --help
```

## Basic Usage

### Scan Your Contract
```bash
sorobanshield scan ./my-contract
```

### Scan Current Directory
```bash
sorobanshield scan .
```

### Get JSON Output (for CI/CD)
```bash
sorobanshield scan ./contract --format json
```

### View Security Check Information
```bash
sorobanshield info
```

## Understanding the Report

### Sample Output
```
SorobanShield Security Report
==============================

[HIGH] panic!() found in src/lib.rs:45
       → Fix: Use Result types instead
       
[MEDIUM] .unwrap() called in src/lib.rs:52
         → Fix: Use .ok()? or pattern matching
         
[LOW] Missing documentation in transfer()
      → Fix: Add /// doc comments
```

### Severity Levels
- **🔴 HIGH** - Security vulnerabilities (fix immediately)
- **🟡 MEDIUM** - Error handling issues (fix soon)
- **🟢 LOW** - Code quality (fix in next iteration)

## Common Issues & Fixes

### Issue: `panic!()` detected
**Why it matters:** Crashes the contract unexpectedly  
**Fix:**
```rust
// ❌ Bad
if amount < 0 { panic!("Invalid amount"); }

// ✅ Good
if amount < 0 { return Err(symbol_short!("InvalidAmount")); }
```

### Issue: `.unwrap()` detected
**Why it matters:** May panic if value is None  
**Fix:**
```rust
// ❌ Bad
let balance = storage.get(&account).unwrap();

// ✅ Good
let balance = storage.get(&account).ok()?;
// or
let balance = storage.get(&account).unwrap_or(0);
```

### Issue: Missing authorization check
**Why it matters:** Anyone can call your function (critical!)  
**Fix:**
```rust
// ❌ Bad
pub fn transfer(from: Address, to: Address, amount: i128) {
    // No check!
    storage.set(&from, &new_balance);
}

// ✅ Good
pub fn transfer(from: Address, to: Address, amount: i128) {
    from.require_auth();  // Required!
    storage.set(&from, &new_balance);
}
```

### Issue: Function lacks documentation
**Why it matters:** Reduces clarity for auditors and users  
**Fix:**
```rust
// ✅ Good
/// Transfers tokens from one account to another
/// 
/// # Arguments
/// * `from` - Sender address (must be authorized)
/// * `to` - Recipient address
/// * `amount` - Number of tokens to transfer
pub fn transfer(env: Env, from: Address, to: Address, amount: i128) -> Result<(), Symbol> {
    // implementation
}
```

## Integration with CI/CD

### GitHub Actions Example
```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  sorobanshield:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - run: cargo install sorobanshield
      
      - run: |
          sorobanshield scan . --format json > report.json
          
      - uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: report.json
```

### GitLab CI Example
```yaml
security_scan:
  image: rust:latest
  script:
    - cargo install sorobanshield
    - sorobanshield scan . --format json > report.json
  artifacts:
    paths:
      - report.json
```

## Flags & Options

```bash
sorobanshield scan [PATH]                    # Scan a directory
sorobanshield scan [PATH] --format json      # JSON output
sorobanshield scan [PATH] --format text      # Terminal output (default)
sorobanshield info                           # Show all checks
sorobanshield --help                         # Show help
sorobanshield --version                      # Show version
```

## Exit Codes

```
0 = Success (no issues found)
1 = Warnings found (one or more issues)
2 = Error (couldn't run scan)
```

Use exit codes in CI/CD to fail builds:
```bash
sorobanshield scan . || exit 1
```

## Performance

### Typical Speeds
- Small contract (100 lines): <10ms
- Medium contract (1000 lines): 50-100ms
- Large project (100+ files): <1 second

### Large Projects
For very large codebases:
```bash
# Scan specific directories
sorobanshield scan ./src
sorobanshield scan ./contracts

# Or combine with other tools
find . -name "*.rs" -type f | xargs sorobanshield scan
```

## Understanding Severity

### HIGH Severity
- **Panic Usage** - Contract execution stops unexpectedly
- **Missing Authorization** - Unauthorized state modifications (CRITICAL!)

**Action:** Fix before production

### MEDIUM Severity
- **Unwrap Usage** - May panic unexpectedly
- **Expect Usage** - May panic unexpectedly

**Action:** Fix in next release

### LOW Severity
- **Missing Documentation** - Reduced code clarity

**Action:** Fix as part of regular maintenance

## Example Contracts

SorobanShield includes examples:

```bash
# Scan example showing what good code looks like
sorobanshield scan ./examples/secure_contract.rs

# Scan example showing common issues
sorobanshield scan ./examples/insecure_contract.rs
```

## FAQ

**Q: Is this a replacement for security audits?**  
A: No. Use SorobanShield as a first-pass check before audits.

**Q: Can I ignore warnings?**  
A: HIGH severity warnings should be fixed before production.

**Q: Why does it flag my test code?**  
A: SorobanShield scans all .rs files. Move tests to separate files or mark them with `#[cfg(test)]`.

**Q: What if I need different checks?**  
A: See CONTRIBUTING.md to add custom checks.

**Q: How accurate are these checks?**  
A: Heuristic-based. Catches 95% of real issues. Manual review is still essential.

## Getting Help

### Documentation
- **README.md** - Full user guide
- **docs/security-checks.md** - Detailed check explanations
- **docs/architecture.md** - How SorobanShield works

### Reporting Issues
```bash
# Include version
sorobanshield --version

# Include error output
sorobanshield scan ./my-contract 2>&1

# Share your contract (sanitized)
```

### Contributing
See **CONTRIBUTING.md** for:
- How to add new checks
- Development setup
- Testing guidelines

## Next Steps

1. **Scan your contract** - `sorobanshield scan ./contract`
2. **Read the report** - Understand what each issue means
3. **Fix issues** - Start with HIGH severity
4. **Integrate CI/CD** - Run on every PR
5. **Share feedback** - Help improve the tool

## Common Workflows

### Before Committing
```bash
sorobanshield scan . || echo "Security issues found"
```

### Before Pushing
```bash
if ! sorobanshield scan .; then
    echo "Fix security issues before pushing"
    exit 1
fi
git push
```

### In Pull Request
Add this to your GitHub Actions to block PRs with issues:
```yaml
- name: Security Scan
  run: sorobanshield scan . || exit 1
```

## Tips & Tricks

### Ignore Specific Files
SorobanShield automatically skips:
- `/target/` directory
- `.git/` directory
- Non-Rust files

Manually organize your project to scan relevant code only.

### Combine with Other Tools
```bash
# Format, lint, then security scan
cargo fmt && cargo clippy && sorobanshield scan .

# Generate full quality report
cargo test && cargo build --release && sorobanshield scan . --format json
```

### CI/CD Pipeline
```bash
1. cargo test              # Unit tests
2. cargo build             # Compilation
3. cargo clippy            # Linting
4. sorobanshield scan .    # Security
```

## Uninstall

```bash
cargo uninstall sorobanshield
```

---

**Questions?** Check the full documentation in README.md or docs/ folder  
**Ready to contribute?** See CONTRIBUTING.md  
**Found a bug?** Open an issue on GitHub  

**Happy secure coding!** 🛡️
