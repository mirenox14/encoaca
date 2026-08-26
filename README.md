
# encoaca

A Rust-based utility and application featuring Nostr protocol integration.

## Project Structure

The repository is structured as a lightweight Rust application:

*   **`main.rs`**: The main entry point of the application.
*   **`key_gen.rs`**: Manages Nostr key pair generation, validation, and local identity storage.
*   **`nostr.rs`**: Handles Nostr protocol interactions, event creation, or relay communication.
*   **`pull.rs`**: Fetches and decrypts environment secrets published under a tag to update `.env`.
*   **`push.rs`**: Encrypts and publishes `.env` secrets under a tag for specified recipient public keys.
*   **`helper.rs`**: Contains shared utility functions and helper methods used throughout the codebase.
*   **`tests.rs`**: Contains unit and integration tests to ensure the reliability of the application's components.
## Prerequisites

To build and run this project, you need to have the Rust toolchain installed on your system. If you do not have it installed, you can set it up via [rustup](https://rustup.rs/):

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
```

### Build the Project

To compile the project and fetch any required dependencies, run:

```bash
cargo build --release
```

### Run the Application

To execute the binary:

```bash
cargo run
```

## Running Tests

The project includes a test suite located in `tests.rs`. You can run the tests using Cargo:

```bash
cargo test
```