# encoaca

A Rust-based application featuring Nostr protocol integration.

## Project Structure

The repository contains the following core files:

*   `main.rs`: The entry point of the application.
*   `nostr.rs`: Module containing Nostr protocol integration and related network or cryptographic utilities.
*   `helper.rs`: Helper functions and internal utilities supporting the codebase.
*   `tests.rs`: Core test suite for validating application logic.

## Prerequisites

To build and run this project, you must have the Rust toolchain installed.

*   [Rust & Cargo](https://www.rust-lang.org/tools/install)

## Getting Started

### Installation

Clone the repository to your local machine:

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
```

### Building the Project

Compile the application using Cargo:

```bash
cargo build --release
```

The compiled binary will be located in the `target/release/` directory.

### Running the Application

To run the binary directly:

```bash
cargo run
```

### Running Tests

Execute the unit and integration tests defined in `tests.rs`:

```bash
cargo test
```