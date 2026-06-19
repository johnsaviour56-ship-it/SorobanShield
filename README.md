# SorobanShield 🛡️

A lightweight security toolkit for Soroban smart contracts. Identify common security mistakes and best-practice violations before deploying.

## Features

- **Fast scanning** - Analyze Soroban contracts in seconds
- **Five security checks** - Covers panic usage, unwrap/expect, missing authorization, and documentation
- **Clear reporting** - Terminal-based output with severity levels and line numbers
- **JSON export** - Integration with CI/CD and audit tools
- **Zero overhead** - No dependencies on heavy frameworks

## Quick Start

### Installation

```bash
cargo install sorobanshield
```

Or build from source:
```bash
git clone https://github.com/johnsaviour56-ship-it/sorobanshield.git
cd sorobanshield
cargo build --release
```

### Basic Usage

```bash
# Scan current directory
sorobanshield scan

# Scan specific contract
sorobanshield scan ./my-contract

# JSON output for CI/CD
sorobanshield scan ./my-contract --format json

# View security check information
sorobanshield info
```

## Example Output

```
======================================================================
SorobanShield Security Report
======================================================================

📁 Files Scanned: 2
📊 Total Lines: 156

⚠️  Warnings Found: 5

HIGH SEVERITY ISSUES:
----------------------------------------------------------------------
  [HIGH] Function 'transfer' may modify state without authorization check at src/lib.rs:45
      Example: pub fn transfer(...) { // Missing require_auth() at start

  [HIGH] panic!() macro found in src/lib.rs:67
      Example: panic!("error message")

MEDIUM SEVERITY ISSUES:
----------------------------------------------------------------------
  [MEDIUM] .unwrap() called in src/lib.rs:52
      Example: some_result.unwrap()

  [MEDIUM] .expect() called in src/lib.rs:88
      Example: some_result.expect("error message")

LOW SEVERITY ISSUES:
----------------------------------------------------------------------
  [LOW] Public function 'calculate' lacks documentation at src/lib.rs:100
      Example: /// This function does X
pub fn func_name() {}

======================================================================
Summary: 2 high 2 medium 1 low
======================================================================
```

## Security Checks

### 1. Panic Usage [HIGH]
Detects `panic!()` macro calls that may halt contract execution unexpectedly.

### 2. Unwrap Usage [MEDIUM]
Detects `.unwrap()` calls that may panic on None/Err values.

### 3. Expect Usage [MEDIUM]
Detects `.expect()` calls that may panic on None/Err values.

### 4. Missing Authorization [HIGH]
Detects public functions that modify state without authorization checks.

### 5. Missing Documentation [LOW]
Detects public functions without doc comments.

For detailed explanations, see [docs/security-checks.md](docs/security-checks.md).

## Project Structure

```
sorobanshield/
├── src/
│   ├── main.rs          # CLI entry point
│   ├── lib.rs           # Library exports
│   ├── scanner.rs       # Scanning orchestration
│   ├── report.rs        # Report formatting
│   ├── models.rs        # Data structures
│   └── checks/
│       ├── mod.rs
│       ├── panic_check.rs
│       ├── unwrap_check.rs
│       ├── expect_check.rs
│       ├── auth_check.rs
│       └── docs_check.rs
├── examples/
│   ├── secure_contract.rs     # Example: passes checks
│   ├── insecure_contract.rs   # Example: fails checks
│   └── README.md
├── docs/
│   └── security-checks.md
└── Cargo.toml
```

## Example Contracts

SorobanShield includes example contracts demonstrating security best practices and common vulnerabilities:

- **secure_contract.rs** - A well-written token contract that passes all checks
- **insecure_contract.rs** - A contract with intentional security issues

Try scanning them to understand what SorobanShield detects:

```bash
sorobanshield scan ./examples
```

## CI/CD Integration

### GitHub Actions

Add to `.github/workflows/security.yml`:

```yaml
name: Security Scan

on: [push, pull_request]

jobs:
  sorobanshield:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install SorobanShield
        run: cargo install sorobanshield
      
      - name: Scan contracts
        run: sorobanshield scan ./contracts --format json > security-report.json
      
      - name: Upload report
        uses: actions/upload-artifact@v3
        with:
          name: security-report
          path: security-report.json
```

### Local Pre-commit Hook

Add to `.git/hooks/pre-commit`:

```bash
#!/bin/bash
sorobanshield scan . || exit 1
```

## FAQ

**Q: Is SorobanShield a replacement for security audits?**
A: No. SorobanShield catches common patterns but cannot substitute for professional audits. Use it as a first-pass defense.

**Q: Can I ignore warnings?**
A: For production, we strongly recommend fixing HIGH severity issues. MEDIUM and LOW issues should be addressed in the next iteration.

**Q: How accurate are the checks?**
A: The checks are heuristic-based and intentionally conservative. False positives are better than missing real issues. Review findings in context.

**Q: Can I add custom checks?**
A: Yes. The `checks/` directory is modular. See the trait in `checks/mod.rs` to implement new checks.

**Q: Does SorobanShield work with all Rust code?**
A: SorobanShield is optimized for Soroban contracts but works on any Rust code. Results will be most useful for smart contract patterns.

## Contributing

Contributions are welcome! Areas for expansion:

- Additional security checks (overflow detection, timestamp validation, etc.)
- Improved heuristics for better detection accuracy
- JSON/HTML report templates
- Plugin system for custom checks
- Language extensions (if Soroban adds other languages)

## License

MIT

## Security Reporting

If you discover a security vulnerability in SorobanShield itself, please email security@stellar.org instead of using the issue tracker.

## Roadmap

### Phase 1 (MVP - Current)
- ✅ Core CLI with 5 checks
- ✅ Terminal and JSON reporting
- ✅ Example contracts
- ✅ Security documentation

### Phase 2
- [ ] Additional checks (integer overflow, timestamp validation, storage patterns)
- [ ] Integration test framework
- [ ] HTML report generation
- [ ] GitHub Action for easy CI/CD integration

### Phase 3
- [ ] LSP support for IDE integration
- [ ] Performance profiling checks
- [ ] Gas optimization suggestions
- [ ] Cross-contract call analysis

## Credits

Built for the Stellar Soroban ecosystem by developers who care about smart contract security.

## Community

- 🐛 [Report Issues](https://github.com/stellar/sorobanshield/issues)
- 💬 [Discuss Ideas](https://github.com/stellar/sorobanshield/discussions)
- 🤝 [Contribute](https://github.com/stellar/sorobanshield)

---

Made with 🛡️ for Soroban developers
