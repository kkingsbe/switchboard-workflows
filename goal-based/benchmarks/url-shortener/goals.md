# Goals

## URL Shortener API

Build a URL shortener HTTP API service in Rust using Axum and SQLite.

### Core Functionality

- POST endpoint to create a short link from a long URL
  - Accepts JSON body with the target URL
  - Returns a unique short code (6-8 alphanumeric characters)
  - Validates that the target URL is a well-formed URL
  - Rejects empty, missing, or malformed URLs with appropriate error responses
  - Optionally accepts a custom short code (reject if already taken)
  - Optionally accepts an expiration duration (e.g., "24h", "7d", "30d")
- GET endpoint that redirects a short code to its target URL
  - Returns 301 redirect to the target URL
  - Returns 404 if the short code doesn't exist
  - Returns 410 Gone if the link has expired
  - Increments a click counter on each successful redirect
- GET endpoint to retrieve link metadata (target URL, created date, click count, expiration)
  - Returns 404 if the short code doesn't exist
- DELETE endpoint to remove a short link
  - Returns 404 if the short code doesn't exist
  - Returns 204 on success

### Data Layer

- SQLite database using sqlx with compile-time checked queries
- Database migrations managed through sqlx migrate
- Links table storing: short code, target URL, created timestamp, expiration timestamp (nullable), click count
- Short code column has a unique constraint
- All database operations use async sqlx, not blocking calls

### Error Handling

- All endpoints return structured JSON error responses with appropriate HTTP status codes
- Validation errors return 400 with a message describing what's wrong
- Internal errors return 500 without leaking implementation details
- No unwrap() or expect() in production code paths — all errors handled via Result types

### Testing

- Unit tests for URL validation logic
- Unit tests for short code generation (uniqueness, length, character set)
- Unit tests for expiration logic (not expired, expired, no expiration set)
- Integration tests for every endpoint using axum::test helpers with an in-memory SQLite database
- Integration tests cover both success and error cases for each endpoint
- All tests pass with `cargo test`

### Project Structure

- Clean module separation: routes, models, database, errors, config
- A lib.rs that exposes the app builder (for testability) separate from main.rs
- Configuration via environment variables (at minimum: DATABASE_URL, HOST, PORT)