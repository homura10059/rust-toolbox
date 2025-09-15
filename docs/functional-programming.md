# Functional Programming Guidelines

This document outlines the functional programming principles and patterns used in this project, based on Scott Wlaschin's "Domain Modeling Made Functional" philosophy.

## Core Philosophy

**IMPORTANT**: This project follows functional programming principles wherever possible, emphasizing domain modeling through types and making illegal states unrepresentable. All new code should be written in functional style with a focus on domain-driven design.

## Domain Modeling Made Functional Principles

### 1. Type-Driven Domain Modeling
- **Make Illegal States Unrepresentable**: Use Rust's type system to encode business rules
- **Domain Types**: Create specific types for domain concepts (ISBN, BookTitle, Price, etc.)
- **Newtype Pattern**: Use tuple structs and validation for domain constraints

```rust
// Good: Domain-specific types using newtype pattern
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct ISBN(String);

#[derive(Debug, Clone, PartialEq, Eq)]
pub struct BookTitle(String);

#[derive(Debug, Clone, PartialEq)]
pub struct Price(f64);

#[derive(Debug, Clone, PartialEq)]
pub struct BookData {
    pub isbn: ISBN,
    pub title: BookTitle,
    pub price: Price,
}
```

### 2. Railway-Oriented Programming
- **Result Types**: Use `Result<T, E>` for operations that can fail
- **Pipeline Composition**: Chain operations using `map`, `and_then`, and combinators
- **Error Handling**: Make errors explicit in the type system

```rust
use regex::Regex;

impl ISBN {
    pub fn parse(input: &str) -> Result<ISBN, String> {
        let re = Regex::new(r"^\d{13}$").unwrap();
        if re.is_match(input) {
            Ok(ISBN(input.to_string()))
        } else {
            Err("Invalid ISBN format".to_string())
        }
    }
}
```

### 3. Pure Functions and Immutability
- **Pure Functions**: Functions should be predictable with no side effects
- **Immutable Data**: Prefer immutable data structures and avoid unnecessary mutations
- **Function Composition**: Break complex logic into small, composable functions

### 4. Pipeline-Oriented Programming
- **Data Transformation Pipelines**: Use function composition to create clear data flow
- **Avoid Imperative Loops**: Use iterators with `map()`, `filter()`, `find()`, `fold()` instead of `for` loops
- **Higher-Order Functions**: Use functions that accept or return other functions

## Domain-Driven Functional Patterns

### Type-Safe Domain Operations

```rust
impl BookTitle {
    pub fn parse(input: &str) -> Result<BookTitle, String> {
        if input.trim().is_empty() {
            Err("Book title cannot be empty".to_string())
        } else {
            Ok(BookTitle(input.trim().to_string()))
        }
    }
}

impl Price {
    pub fn parse(input: f64) -> Result<Price, String> {
        if input < 0.0 {
            Err("Price cannot be negative".to_string())
        } else {
            Ok(Price(input))
        }
    }
}

// Good: Domain-specific operations with type safety
pub fn create_book_data(
    isbn: &str,
    title: &str,
    price: f64,
) -> Result<BookData, String> {
    let isbn = ISBN::parse(isbn)?;
    let title = BookTitle::parse(title)?;
    let price = Price::parse(price)?;
    
    Ok(BookData { isbn, title, price })
}
```

### Pipeline-Oriented Data Processing

```rust
// Good: Pipeline-oriented selector matching using iterators
fn find_book_value(selectors: &[&str]) -> Option<String> {
    selectors
        .iter()
        .map(|selector| scrape_text(selector).trim().to_string())
        .find(|text| !text.is_empty())
}

// Avoid: Imperative loops with mutations
// let mut value = String::new();
// for selector in selectors {
//     value = scrape_text(selector).trim().to_string();
//     if !value.is_empty() {
//         break;
//     }
// }
```

### Error Railway Pattern

```rust
#[derive(Debug)]
pub enum ScrapingError {
    NetworkError(String),
    ParseError(String),
    ValidationError(String),
}

// Chain operations that can fail using ? operator
pub fn scrape_book_details(url: &str) -> Result<BookData, ScrapingError> {
    let page = fetch_page(url)?;
    let isbn_str = extract_isbn(&page)?;
    let title_str = extract_title(&page)?;
    let price_value = extract_price(&page)?;
    
    let isbn = ISBN::parse(&isbn_str)
        .map_err(|e| ScrapingError::ValidationError(e))?;
    let title = BookTitle::parse(&title_str)
        .map_err(|e| ScrapingError::ValidationError(e))?;
    let price = Price::parse(price_value)
        .map_err(|e| ScrapingError::ValidationError(e))?;
    
    Ok(BookData { isbn, title, price })
}
```

## Domain Types and Constraints

### Newtype Pattern for Domain Safety

```rust
use url::Url;

// Create domain-specific types that prevent mixing
#[derive(Debug, Clone, PartialEq)]
pub struct SafeURL(Url);

#[derive(Debug, Clone, PartialEq)]
pub struct Selector(String);

#[derive(Debug, Clone, PartialEq)]
pub struct ScrapedText(String);

// Factory functions with validation
impl SafeURL {
    pub fn parse(input: &str) -> Result<SafeURL, String> {
        Url::parse(input)
            .map(SafeURL)
            .map_err(|e| format!("Invalid URL format: {}", e))
    }
}
```

### State Machines with Types

```rust
// Make scraping states explicit using enums
#[derive(Debug, Clone, PartialEq)]
pub enum ScrapingState {
    Idle,
    Fetching { url: SafeURL },
    Parsing { html: String },
    Completed { data: BookData },
    Failed { error: ScrapingError },
}

#[derive(Debug, Clone, PartialEq)]
pub enum ScrapingEvent {
    StartFetch { url: SafeURL },
    PageFetched { html: String },
    DataExtracted { data: BookData },
    ErrorOccurred { error: ScrapingError },
}

// State transitions are explicit and type-safe
pub fn transition(state: ScrapingState, event: ScrapingEvent) -> ScrapingState {
    match (state, event) {
        (ScrapingState::Idle, ScrapingEvent::StartFetch { url }) => {
            ScrapingState::Fetching { url }
        }
        (ScrapingState::Fetching { .. }, ScrapingEvent::PageFetched { html }) => {
            ScrapingState::Parsing { html }
        }
        (ScrapingState::Parsing { .. }, ScrapingEvent::DataExtracted { data }) => {
            ScrapingState::Completed { data }
        }
        (_, ScrapingEvent::ErrorOccurred { error }) => {
            ScrapingState::Failed { error }
        }
        // Return current state for invalid transitions
        (state, _) => state,
    }
}
```

## Implementation Guidelines

### Refactoring to Domain-Driven Functional Style

1. **Identify Domain Concepts**: Convert primitive types to newtype wrappers
2. **Make Invalid States Impossible**: Use enums and type constraints
3. **Create Validation Pipelines**: Use Result types for operations that can fail
4. **Compose Functions**: Build complex operations from simple, pure functions
5. **Handle Errors Explicitly**: Use Result and Option types consistently

### Testing Domain Functions

```rust
#[cfg(test)]
mod tests {
    use super::*;

    // Test domain functions with clear assertions
    #[test]
    fn should_accept_valid_13_digit_isbns() {
        let result = ISBN::parse("9784274217884");
        assert!(result.is_ok());
        assert_eq!(result.unwrap().0, "9784274217884");
    }
    
    #[test]
    fn should_reject_invalid_isbn_formats() {
        let result = ISBN::parse("invalid");
        assert!(result.is_err());
        assert!(result.unwrap_err().contains("Invalid ISBN format"));
    }
    
    // Property-based testing with proptest
    use proptest::prelude::*;
    
    proptest! {
        #[test]
        fn valid_isbn_always_parseable(s in r"\d{13}") {
            prop_assert!(ISBN::parse(&s).is_ok());
        }
    }
}
```

## Benefits of Domain Modeling Made Functional

- **Type Safety**: Domain types prevent runtime errors and invalid states
- **Self-Documenting Code**: Types express business rules and domain concepts
- **Composability**: Small, focused functions can be easily combined into pipelines
- **Error Handling**: Explicit error types make failure cases visible and handleable
- **Maintainability**: Changes to domain rules are reflected in the type system
- **Testability**: Pure functions and explicit types make testing straightforward
- **Refactoring Safety**: Type system catches breaking changes during refactoring