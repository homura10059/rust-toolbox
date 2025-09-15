# Test-Driven Development (TDD)

This document outlines the Test-Driven Development practices for this project, following t-wada's (和田卓人) recommended approach.

## TDD Philosophy

TDD is fundamentally about **developer testing** that creates testable designs and reduces development anxiety through incremental success. The key principle is to prioritize "usability" over "buildability" in design.

## TDD Workflow (t-wada's 5-Step Process)

Following Kent Beck's definition as advocated by t-wada:

1. **Write a comprehensive test scenario list** - Plan what needs to be tested
2. **Select one item and write a failing test** - Focus on one test at a time, ensure it fails initially
3. **Write minimal code to pass** - Implement just enough code to make the current test (and all previous tests) pass
4. **Refactor as needed** - Improve implementation design while maintaining test success
5. **Return to step 2** - Repeat until the test list is empty

### Core Principles

- **One test at a time** - Never write multiple tests simultaneously
- **Small incremental steps** - Build confidence through gradual progress
- **Same timing** - Write tests in the same timing as implementation
- **User perspective** - Tests should verify expected behavior from the user's viewpoint

## Testing Commands

- `cargo test` - Run all tests in workspace
- `cargo test -p <crate-name>` - Run tests for specific crate
- `cargo test --all` - Run all tests across workspace
- `cargo test -- --nocapture` - Show print statements in tests
- `cargo watch -x test` - Run tests in watch mode (requires cargo-watch)

## TDD Benefits (t-wada's Insights)

- **Immediate feedback** on code functionality
- **Reduced fear** of changing existing code
- **Systematic problem-solving** for developers
- **Confidence building** through incremental success
- **Task management** and problem decomposition skills

## Testing Guidelines

Following t-wada's recommendations:

- **Write tests before implementation** - Tests define requirements and expected behavior
- **Focus on one test at a time** - Avoid writing multiple tests simultaneously
- **Keep tests simple** - Each test should verify one specific piece of functionality
- **Test independence** - Each test should be independent of others
- **Descriptive test names** - Test names should explain the behavior being verified
- **User perspective** - Tests should verify expected behavior from the user's viewpoint
- **Mock external dependencies** - HTTP requests, file system operations, etc.
- **Group related tests** using `describe` blocks
- **Aim for high coverage** (>80%) focusing on critical business logic

## Test File Structure

- Place unit tests in the same file using `#[cfg(test)]` modules
- Place integration tests in `tests/` directory at crate root
- Follow Rust conventions: `src/lib.rs` with embedded tests or `tests/integration_test.rs`

## Learning TDD (t-wada's Approach)

t-wada recommends starting TDD learning through:

1. **Imitation and "copying" (写経)** - Practice through hands-on exercises
2. **Tutorial-based learning** - Follow structured examples
3. **Individual practice** - TDD can be practiced independently, even in waterfall projects
4. **Experience psychological benefits** - Feel the increased confidence and reduced anxiety

## Practical Implementation

### Step-by-Step Example

```rust
// 1. Write test scenario list (TODO)
// - fetch_book_data should return book data for valid ISBN
// - fetch_book_data should return error for invalid ISBN
// - fetch_book_data should handle network errors

#[derive(Debug, PartialEq)]
pub struct BookData {
    pub title: String,
    pub author: String,
    pub isbn: String,
}

pub struct ScraperService;

impl ScraperService {
    pub fn new() -> Self {
        Self
    }
    
    // 3. Write minimal code to pass
    pub fn fetch_book_data(&self, isbn: &str) -> Result<BookData, String> {
        Ok(BookData {
            title: "テスト駆動開発".to_string(),
            author: "Kent Beck".to_string(),
            isbn: "9784274217884".to_string(),
        })
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    // 2. Select one item and write failing test
    #[test]
    fn should_return_book_data_for_valid_isbn() {
        let service = ScraperService::new();
        let result = service.fetch_book_data("9784274217884");
        
        assert_eq!(result.unwrap(), BookData {
            title: "テスト駆動開発".to_string(),
            author: "Kent Beck".to_string(),
            isbn: "9784274217884".to_string(),
        });
    }
    
    // 4. Refactor as needed
    // 5. Return to step 2 for next test
}
```

### Mocking External Dependencies

```rust
// For HTTP client mocking, use mockito or wiremock
#[cfg(test)]
mod tests {
    use mockito::Server;
    
    #[tokio::test]
    async fn test_with_mock_server() {
        let mut server = Server::new_async().await;
        let mock = server
            .mock("GET", "/book/9784274217884")
            .with_status(200)
            .with_body(r#"{"title":"テスト駆動開発"}"#)
            .create_async()
            .await;
        
        // Test implementation here
        mock.assert_async().await;
    }
}
```

### Test Coverage

- Focus on **critical business logic** and edge cases
- Use `cargo tarpaulin` for coverage reporting
- Run with `cargo tarpaulin --out html` to generate HTML coverage reports
- Maintain >80% coverage while prioritizing meaningful tests over metrics