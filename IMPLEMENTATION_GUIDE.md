# SorobanShield MVP Implementation Guide

This guide walks through building and testing SorobanShield from scratch.

## Prerequisites

- **Rust 1.70+** - Install from https://rustup.rs/
- **Git** - Version control
- **Text Editor** - VS Code recommended

## Installation & Setup

### 1. Install Rust
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
rustc --version  # Verify installation
```

### 2. Clone the Repository
```bash
git clone https://github.com/johnsaviour56-ship-it/sorobanshield.git
cd sorobanshield
```

### 3. Build the Project
```bash
# Development build
cargo build

# Release build (optimized)
cargo build --release
```

### 4. Verify Installation
```bash
# Run from source
cargo run -- info

# Or run binary directly
./target/debug/sorobanshield info
```

## Running Tests

### All Tests
```bash
cargo test
```

### Specific Check Tests
```bash
# Test panic check only
cargo test panic_check::tests

# Test with output
cargo test -- --nocapture
```

### Test Coverage
```bash
# Install tarpaulin
cargo install cargo-tarpaulin

# Generate coverage report
cargo tarpaulin --out Html
```

## Using SorobanShield

### Scan a Directory
```bash
# Scan current directory
sorobanshield scan

# Scan specific contract
sorobanshield scan ./my-contract

# Scan with JSON output
sorobanshield scan ./my-contract --format json > report.json
```

### View Check Information
```bash
sorobanshield info
```

### Example Scanning
```bash
# Scan the provided examples
sorobanshield scan ./examples

# You should see:
# - 0 issues in secure_contract.rs
# - Multiple issues in insecure_contract.rs
```

## Development Workflow

### Adding a New Check

#### 1. Create File
```bash
touch src/checks/your_check.rs
```

#### 2. Implement Check
See `src/checks/panic_check.rs` for template. Basic structure:

```rust
pub struct YourCheck;

impl super::SecurityCheck for YourCheck {
    fn name(&self) -> &'static str {
        "your_check_name"
    }

    fn check(&self, file_path: &PathBuf, content: &str) -> Vec<Finding> {
        // Implementation
    }
}
```

#### 3. Register Check
Edit `src/checks/mod.rs`:

```rust
pub fn run_all_checks(file_path: &PathBuf, content: &str) -> Vec<Finding> {
    let checks: Vec<Box<dyn SecurityCheck>> = vec![
        // ... existing checks ...
        Box::new(your_check::YourCheck),  // Add your check
    ];
    // ...
}
```

#### 4. Add Tests
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_detects_issue() {
        let check = YourCheck;
        let code = r#"code with issue"#;
        let findings = check.check(&PathBuf::from("test.rs"), code);
        assert!(!findings.is_empty());
    }
}
```

#### 5. Test Everything
```bash
cargo test
cargo build --release
./target/release/sorobanshield scan ./examples
```

## Code Quality

### Format Code
```bash
cargo fmt
```

### Check for Issues
```bash
cargo clippy
```

### Fix Warnings
```bash
cargo clippy -- -D warnings
```

## Performance Testing

### Benchmark Small Contract
```bash
time cargo run --release -- scan ./examples
```

### Benchmark Large Directory
```bash
# Create test directory with many files
mkdir -p test_data
for i in {1..100}; do cp examples/secure_contract.rs test_data/contract_$i.rs; done

# Time the scan
time cargo run --release -- scan ./test_data
```

Expected: Should scan 100 files (10k+ lines) in <1 second

## Packaging and Distribution

### Build Release Binary
```bash
cargo build --release
# Binary located at: target/release/sorobanshield
```

### Install Locally
```bash
cargo install --path .
# Now available as 'sorobanshield' in any terminal
```

### Publish to Crates.io
```bash
# Update Cargo.toml with version
# Create CHANGELOG.md entry

cargo publish
```

## CI/CD Integration

### GitHub Actions
Create `.github/workflows/ci.yml`:

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
      - run: cargo test --verbose
      - run: cargo clippy -- -D warnings
      - run: cargo fmt -- --check

  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions-rs/toolchain@v1
        with:
          toolchain: stable
      - run: cargo build --release
      - run: ./target/release/sorobanshield scan ./examples
```

## Troubleshooting

### Build Fails
```bash
# Clean and rebuild
cargo clean
cargo build

# Check Rust version
rustc --version
# Should be 1.70+
```

### Tests Fail
```bash
# Run with verbose output
cargo test -- --nocapture

# Run specific test
cargo test test_name -- --nocapture

# Check for environment issues
echo $RUST_BACKTRACE
export RUST_BACKTRACE=1
cargo test
```

### Slow Performance
```bash
# Use release build for testing
cargo run --release -- scan ./contract

# Profile with flamegraph
cargo install flamegraph
cargo flamegraph
# Generates flamegraph.svg
```

## Documentation

### Update README
```bash
# Edit README.md
# Use markdown formatting
# Include code examples
# Test all links
```

### Add Check Documentation
```bash
# Edit docs/security-checks.md
# Add section for new check
# Include vulnerable example
# Include recommended fix
# Add test case reference
```

### Update Architecture
```bash
# Edit docs/architecture.md
# Update diagrams if needed
# Document new components
# Add performance notes
```

## Release Checklist

Before releasing a new version:

```bash
# 1. Update version in Cargo.toml
# 2. Update CHANGELOG.md
# 3. Run full test suite
cargo test

# 4. Build release binary
cargo build --release

# 5. Test on examples
./target/release/sorobanshield scan ./examples

# 6. Verify no warnings
cargo clippy
cargo fmt

# 7. Create git tag
git tag v0.2.0
git push origin v0.2.0

# 8. Publish to crates.io
cargo publish
```

## Project Statistics

### Code Metrics
```bash
# Count lines of code
find src -name "*.rs" | xargs wc -l

# List dependencies
cargo tree
```

### Example Output
```
src/main.rs:              ~150 lines
src/lib.rs:               ~10 lines
src/scanner.rs:           ~70 lines
src/report.rs:            ~130 lines
src/models.rs:            ~90 lines
src/checks/mod.rs:        ~30 lines
src/checks/panic_check.rs: ~60 lines
src/checks/unwrap_check.rs: ~60 lines
src/checks/expect_check.rs: ~60 lines
src/checks/auth_check.rs:  ~90 lines
src/checks/docs_check.rs:  ~80 lines

Total: ~820 lines of production code
+     ~600 lines of tests and examples
= ~1420 lines total
```

## Next Steps After MVP

### Phase 2 Goals
- [ ] Add 2-3 additional security checks
- [ ] Implement HTML report generation
- [ ] Create GitHub Action for easy CI/CD
- [ ] Improve auth check heuristic accuracy
- [ ] Add integration test framework

### Phase 3 Goals
- [ ] LSP support for IDE integration
- [ ] AST-based analysis for accuracy
- [ ] Parallel scanning for large projects
- [ ] Plugin system for custom checks
- [ ] Cross-contract analysis

### Community
- [ ] Collect feedback from developers
- [ ] Create issue templates
- [ ] Establish contribution guidelines
- [ ] Build community examples
- [ ] Create YouTube tutorial

## Maintenance

### Regular Tasks
```bash
# Update dependencies
cargo update

# Check for security vulnerabilities
cargo audit

# Check for outdated dependencies
cargo outdated
```

### Monitoring
- Check GitHub issues regularly
- Review PRs within 24 hours
- Keep documentation up-to-date
- Monitor Soroban ecosystem changes

---

## Quick Reference

### Common Commands
```bash
cargo build              # Build debug binary
cargo build --release   # Build optimized binary
cargo test              # Run all tests
cargo run -- scan .     # Run scanner on current directory
cargo fmt               # Format code
cargo clippy            # Check for issues
cargo doc --open        # Generate and view documentation
```

### Project Structure Quick View
```
src/
  ├── main.rs           # Entry point
  ├── scanner.rs        # File discovery & orchestration
  ├── checks/           # Security checks
  ├── models.rs         # Data structures
  └── report.rs         # Output formatting

examples/
  ├── secure_contract.rs    # Best practices
  └── insecure_contract.rs  # Anti-patterns

docs/
  ├── security-checks.md    # Check documentation
  └── architecture.md       # System design
```

### File Extensions Reference
- `.rs` - Rust source files
- `.toml` - Configuration files
- `.md` - Markdown documentation
- `.json` - JSON output and reports

---

Happy coding! 🛡️
