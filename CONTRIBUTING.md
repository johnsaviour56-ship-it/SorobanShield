# Contributing to SorobanShield

Thank you for your interest in contributing! This document explains how to get started.

## Getting Started

### Prerequisites
- Rust 1.70+ (install from https://rustup.rs/)
- Git
- A text editor of your choice

### Setup
```bash
git clone https://github.com/johnsaviour56-ship-it/sorobanshield.git
cd sorobanshield
cargo build
cargo test
```

## Development Workflow

### 1. Create a Feature Branch
```bash
git checkout -b feature/my-feature
```

### 2. Make Your Changes
Follow the code style and patterns in the project.

### 3. Test Your Changes
```bash
# Run all tests
cargo test

# Run with logging
RUST_LOG=debug cargo test

# Build release binary
cargo build --release
```

### 4. Commit with Clear Messages
```bash
git commit -m "feat: add new security check for X"
```

Use conventional commit format:
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation
- `test:` for test improvements
- `refactor:` for code improvements

### 5. Push and Create a Pull Request
```bash
git push origin feature/my-feature
```

## Adding a New Security Check

### Step 1: Create the Check File
```bash
touch src/checks/mycheck_check.rs
```

### Step 2: Implement the Check Trait
```rust
use crate::models::{Finding, Severity};
use regex::Regex;
use std::path::PathBuf;

pub struct MyCheck;

impl super::SecurityCheck for MyCheck {
    fn name(&self) -> &'static str {
        "my_check"
    }

    fn check(&self, file_path: &PathBuf, content: &str) -> Vec<Finding> {
        let mut findings = Vec::new();
        let pattern = Regex::new(r"your_pattern").unwrap();
        
        for (line_num, line) in content.lines().enumerate() {
            if pattern.is_match(line) {
                // Skip comments
                if let Some(comment_pos) = line.find("//") {
                    if pattern.find(&line[..comment_pos]).is_none() {
                        continue;
                    }
                }
                
                let finding = Finding::new(
                    Severity::Medium,
                    self.name(),
                    file_path.clone(),
                    line_num + 1,
                    "Your finding message".to_string(),
                );
                findings.push(finding);
            }
        }
        
        findings
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_detects_issue() {
        let check = MyCheck;
        let code = r#"
// Your test code
        "#;
        let findings = check.check(&PathBuf::from("test.rs"), code);
        assert!(!findings.is_empty());
    }

    #[test]
    fn test_ignores_in_comment() {
        let check = MyCheck;
        let code = r#"
// // your pattern in comment
        "#;
        let findings = check.check(&PathBuf::from("test.rs"), code);
        assert!(findings.is_empty());
    }
}
```

### Step 3: Register the Check
Edit `src/checks/mod.rs` and add to `run_all_checks()`:

```rust
pub fn run_all_checks(file_path: &PathBuf, content: &str) -> Vec<Finding> {
    let checks: Vec<Box<dyn SecurityCheck>> = vec![
        Box::new(panic_check::PanicCheck),
        Box::new(unwrap_check::UnwrapCheck),
        Box::new(expect_check::ExpectCheck),
        Box::new(auth_check::AuthCheck),
        Box::new(docs_check::DocsCheck),
        Box::new(mycheck_check::MyCheck),  // ADD HERE
    ];

    let mut findings = Vec::new();
    for check in checks {
        findings.extend(check.check(file_path, content));
    }
    findings
}
```

### Step 4: Document the Check
Edit `docs/security-checks.md` and add a section explaining:
- What it detects
- Why it matters
- Vulnerable code example
- Recommended fix
- When it's safe to ignore

### Step 5: Test Thoroughly
```bash
cargo test
./target/debug/sorobanshield scan ./examples
```

## Code Style

### Formatting
```bash
cargo fmt
```

### Linting
```bash
cargo clippy
```

### Guidelines
- Use clear variable names
- Add comments for complex logic
- Keep functions focused and small
- Prefer explicit error handling over panics
- Use type hints where unclear

## Testing Guidelines

### Unit Tests
Each check should have unit tests for:
- **Positive case** - The issue is detected
- **Comment case** - Comments are ignored
- **Edge case** - Boundary conditions

### Test Structure
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_name_describes_what_is_tested() {
        // Arrange
        let check = MyCheck;
        let code = r#"problematic code"#;
        
        // Act
        let findings = check.check(&PathBuf::from("test.rs"), code);
        
        // Assert
        assert!(!findings.is_empty());
        assert_eq!(findings[0].severity, Severity::Medium);
    }
}
```

## Documentation

### README Updates
If your change affects how SorobanShield is used, update the README.

### Code Comments
Use comments for:
- **Why** (not what) - Explain the reasoning
- Complex algorithms
- Non-obvious design decisions

### Doc Comments
Use `///` doc comments for:
- All public functions
- Trait methods
- Complex structs

## Performance Considerations

### Regex Efficiency
- Compile patterns once at module level
- Use `.is_match()` before `.find()` for boolean checks
- Avoid backtracking with complex patterns

### Memory
- Stream file processing (don't load entire directory into memory)
- Reuse vectors where possible
- Be mindful of large files

### Benchmarking
Test your check with a large contract:
```bash
time cargo run --release -- scan /path/to/large/contract
```

## Pull Request Process

1. **Create PR with clear title** - Use conventional format
2. **Write description** - What changed and why
3. **Link issues** - Reference related GitHub issues
4. **Pass CI** - All tests must pass
5. **Request review** - Wait for project maintainer review

### PR Description Template
```markdown
## Description
Brief explanation of the change.

## Motivation
Why is this change needed?

## Testing
How was this tested?

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Code follows project style
- [ ] No breaking changes
```

## Release Process

The maintainers handle releases. Merged PRs will be included in the next release.

Version format: `MAJOR.MINOR.PATCH`
- MAJOR: Breaking changes
- MINOR: New features (backward compatible)
- PATCH: Bug fixes

## Community Guidelines

- Be respectful and constructive
- Ask questions if anything is unclear
- Share ideas in discussions before large changes
- Credit contributors (mentioned in CHANGELOG)

## Getting Help

- 📖 Check existing documentation
- 🔍 Search existing issues
- 💬 Ask in GitHub discussions
- 🐛 File an issue for bugs

## Code of Conduct

By contributing, you agree to uphold our community standards:
- Treat all contributors with respect
- Welcome diverse perspectives
- Focus on constructive feedback
- Report violations to maintainers

---

Thank you for making SorobanShield better! 🛡️
