# encoaca

A lightweight, Rust-based application integrating with the Nostr protocol.

This repository contains the core logic for a Nostr-compatible utility, structured with modular helper functions, specialized Nostr protocol integration, and a dedicated testing suite.

## Project Structure

The project is organized into the following Rust modules:

*   **`main.rs`**: The entry point of the application, coordinating the initialization and execution flow.
*   **`nostr.rs`**: Handles Nostr protocol operations, event creation, signing, or communication.
*   **`helper.rs`**: Provides auxiliary utility functions and shared helpers used throughout the application.
*   **`tests.rs`**: Contains unit and integration tests to ensure code reliability and correctness.

## Prerequisites

To build and run this project, you need to have the Rust toolchain installed:

*   **Rust** (edition 2021 or later recommended)
*   **Cargo** (Rust's package manager)

Install Rust via [rustup](https://rustup.rs/):

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
```

### 2. Build the Project

Compile the application using Cargo:

```bash
cargo build --release
```

### 3. Run the Application

Execute the compiled binary:

```bash
cargo run
```

### 4. Running Tests

Validate the implementation by running the test suite defined in `tests.rs`:

```bash
cargo test
```