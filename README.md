# encoaca

`encoaca` is a Rust-based application designed to interface with the Nostr protocol. It provides a modular structure for connecting to Nostr relays, handling events, and executing utility helpers.

## Features

- **Nostr Protocol Integration**: Dedicated module (`nostr.rs`) for interacting with Nostr relays and managing cryptographic keys or events.
- **Utility Helpers**: Modular helper functions (`helper.rs`) to streamline internal operations.
- **Robust Testing**: Comprehensive unit and integration test suite located in `tests.rs`.

## Project Structure

```text
├── helper.rs      # Helper functions and common utilities
├── main.rs        # Application entry point
├── nostr.rs       # Nostr client logic and protocol implementation
├── tests.rs       # Unit and integration tests
└── README.md      # Project documentation
```

## Prerequisites

To build and run this project, you must have the Rust toolchain installed on your system. If you do not have Rust installed, you can install it via [rustup](https://rustup.rs/):

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Getting Started

### Clone the Repository

```sh
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
```

### Build the Application

To build the project in release mode:

```sh
cargo build --release
```

### Run the Application

To execute the binary:

```sh
cargo run
```

## Running Tests

The project includes unit and integration tests to verify functionality. To run the test suite:

```sh
cargo test
```