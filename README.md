# encoaca

A high-performance, secure Rust daemon and library designed for local caching, cryptographic processing, and proxying of Nostr protocol events. 

[![Build Status](https://img.shields.io/github/actions/workflow/status/mirenox14/encoaca/ci.yml?branch=main&style=flat-square)](https://github.com/mirenox14/encoaca/actions)
[![Crates.io](https://img.shields.io/crates/v/encoaca.svg?style=flat-square)](https://crates.io/crates/encoaca)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg?style=flat-square)](LICENSE)
[![Nostr Protocol](https://img.shields.io/badge/nostr-protocol-purple.svg?style=flat-square)](https://github.com/nostr-protocol/nips)

`encoaca` (Encrypted Nostr Content Archiver & Caching Agent) bridges the gap between decentralized Nostr relays and resource-constrained local applications. By acting as a local caching proxy and cryptographic offloader, it handles heavy operations—such as NIP-44/NIP-04 decryption, signature verification, and complex event filtering—closer to the client, ensuring sub-millisecond query times and offline resilience.

---

## Features

- **High-Performance Event Caching**: In-memory and persistent storage of verified Nostr events to minimize relay roundtrips.
- **Cryptographic Offloading**: Multi-threaded signature verification and payload decryption using optimized Rust crypto primitives (`secp256k1`, `chacha20poly1305`).
- **NIP Support**:
  - **NIP-01**: Basic protocol flow, event signing, and verification.
  - **NIP-04**: Legacy Encrypted Direct Messages.
  - **NIP-19**: Bech32-encoded entities (`npub`, `nsec`, `note`, `nevent`).
  - **NIP-44**: Modern, secure versioned encryption.
- **Resilient Connection Management**: Automatic reconnection, backoff strategies, and health monitoring for multiple upstream relays.
- **Advanced Filtering**: Local subscription management with complex JSON-based query filters.

---

## Tech Stack

- **Language**: Rust (Edition 2021)
- **Asynchronous Runtime**: [Tokio](https://tokio.rs/)
- **Nostr Integration**: [nostr](https://crates.io/crates/nostr) & [nostr-database](https://crates.io/crates/nostr-database)
- **Cryptography**: `secp256k1`, `aes-gcm`, `chacha20poly1305`
- **Serialization**: `serde`, `serde_json`
- **Logging & Diagnostics**: `tracing` & `tracing-subscriber`

---

## Architecture

The project is structured modularly to separate protocol handling, cryptographic operations, and system orchestration:

```
├── README.md
├── helper.rs       # Cryptographic utilities, key derivation, and file I/O helpers
├── main.rs         # CLI entrypoint, runtime initialization, and event loop
├── nostr.rs        # Relay management, subscription handling, and NIP implementations
└── tests.rs        # Integration and unit test suites
```

### Component Interaction Flow

```
┌─────────────────┐       Event Stream       ┌──────────────┐
│  Nostr Relays   │ ───────────────────────> │   nostr.rs   │ (Connection & Sub Management)
└─────────────────┘                          └──────┬───────┘
                                                    │ Raw Events
                                                    v
┌─────────────────┐     Decrypted/Verified   ┌──────────────┐
│ Local Client/UI │ <─────────────────────── │  helper.rs   │ (Signature Verification,
└─────────────────┘                          └──────────────┘  NIP-44 Decryption)
```

---

## Getting Started

### Prerequisites

Ensure you have the following installed:
- **Rust Toolchain**: `rustc` and `cargo` (v1.75.0 or higher recommended)
- **OpenSSL**: Required for secure WebSocket connections (or use the `rustls` feature flags)

### Installation

#### From Source

```bash
# Clone the repository
git clone https://github.com/mirenox14/encoaca.git
cd encoaca

# Build in release mode
cargo build --release

# Verify installation
./target/release/encoaca --version
```

---

## Configuration

`encoaca` can be configured using environment variables or a `.env` file in the working directory.

```env
# Logging Level (error, warn, info, debug, trace)
RUST_LOG=info

# Nostr Identity (Hex or Bech32 nsec)
NOSTR_PRIVATE_KEY=nsec1...

# Upstream Relays (Comma-separated)
NOSTR_RELAYS=wss://relay.damus.io,wss://nos.lol,wss://relay.nostr.band

# Local Cache Path
ENCOACA_CACHE_DIR=./.cache/encoaca

# Local Bind Address for Client API
ENCOACA_BIND_ADDR=127.0.0.1:8080
```

---

## Usage

### Running the Daemon

To start the local caching proxy:

```bash
# Run with default configuration
cargo run --release -- daemon

# Run with custom relays and debug logging
RUST_LOG=debug cargo run --release -- daemon --relays wss://relay.damus.io wss://nos.lol
```

### Programmatic Usage

You can import `encoaca` modules into your own Rust projects.

#### Decrypting a NIP-44 Event

```rust
use encoaca::helper::{decrypt_nip44, derive_shared_secret};
use nostr::prelude::*;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let sender_keys = Keys::generate();
    let receiver_keys = Keys::generate();

    let message = "Confidential Nostr payload";
    
    // Encrypt using NIP-44
    let encrypted_content = nip44::encrypt(
        &sender_keys.secret_key()?,
        &receiver_keys.public_key(),
        message,
        nip44::Version::V2
    )?;

    // Decrypt using helper module
    let decrypted = decrypt_nip44(
        &receiver_keys.secret_key()?,
        &sender_keys.public_key(),
        &encrypted_content
    )?;

    assert_eq!(message, decrypted);
    println!("Successfully decrypted: {}", decrypted);
    Ok(())
}
```

#### Subscribing to Relays

```rust
use encoaca::nostr::NostrClient;
use nostr::prelude::*;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = NostrClient::new(vec![
        "wss://relay.damus.io".to_string(),
    ]).await?;

    // Subscribe to text notes (Kind 1)
    let filter = Filter::new().kind(Kind::TextNote).limit(10);
    client.subscribe(vec![filter]).await?;

    // Process incoming events
    client.handle_events(|event| {
        println!("Received Event ID: {}", event.id);
        Ok(())
    }).await?;

    Ok(())
}
```

---

## Development

### Running Tests

The test suite covers cryptographic helpers, relay subscription flows, and event serialization.

```bash
# Run all tests
cargo test

# Run tests with stdout printing enabled
cargo test -- --nocapture

# Run specific integration tests
cargo test test_nip44_encryption_flow
```

### Formatting and Linting

Ensure code quality adheres to Rust standards:

```bash
# Format check
cargo fmt --all -- --check

# Run Clippy lints
cargo clippy --all-targets --all-features -- -D warnings
```

---

## Deployment

### Docker Deployment

A lightweight Docker configuration is available for containerized environments.

```bash
# Build the Docker image
docker build -t mirenox14/encoaca:latest .

# Run the container
docker run -d \
  --name encoaca \
  -e RUST_LOG=info \
  -e NOSTR_RELAYS=wss://relay.damus.io \
  -p 8080:8080 \
  mirenox14/encoaca:latest
```

### Systemd Service

To run `encoaca` as a persistent background daemon on Linux:

1. Create a service file `/etc/systemd/system/encoaca.service`:

```ini
[Unit]
Description=encoaca Nostr Caching Daemon
After=network.target

[Service]
Type=simple
User=encoaca
WorkingDirectory=/var/lib/encoaca
EnvironmentFile=/etc/encoaca/config.env
ExecStart=/usr/local/bin/encoaca daemon
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

2. Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now encoaca
```

---

## Troubleshooting

### Common Issues

#### 1. OpenSSL Build Failures
If you encounter compilation errors related to OpenSSL:
- **macOS**: Run `brew install openssl` and set:
  ```bash
  export OPENSSL_ROOT_DIR="/opt/homebrew/opt/openssl@3"
  ```
- **Ubuntu/Debian**: Install development headers:
  ```bash
  sudo apt-get install pkg-config libssl-dev
  ```

#### 2. Connection Drops to Relays
If the daemon frequently disconnects:
- Check your network connection.
- Ensure the relays specified in `NOSTR_RELAYS` are active using a tool like [nostr.watch](https://nostr.watch).
- Increase the connection timeout in your configuration.

---

## Roadmap

- [ ] **Persistent Database Backend**: Support for RocksDB and SQLite for robust local event storage.
- [ ] **WebAssembly (WASM) Target**: Compile the helper module to WASM for browser-side execution.
- [ ] **NIP-42 Authentication**: Support for relay authentication challenges.
- [ ] **GraphQL Interface**: Expose a local GraphQL endpoint for complex client queries.

---

## License

This project is dual-licensed under:
- **MIT License** ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)
- **Apache License, Version 2.0** ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)

---

## Contributors & Acknowledgments

- **mirenox14** - Creator and Lead Maintainer
- Special thanks to the [rust-nostr](https://github.com/rust-nostr/nostr) community for their foundational crates.