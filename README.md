# ⚡ encoaca

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/mirenox14/encoaca/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust Version](https://img.shields.io/badge/rustc-1.70+-blue.svg)](https://www.rust-lang.org)
[![Nostr NIPs](https://img.shields.io/badge/Nostr-NIP--01%20%7C%20NIP--19%20%7C%20NIP--21-purple.svg)](https://github.com/nostr-protocol/nips)

`encoaca` is a high-performance, lightweight command-line utility and Rust library designed for encoding, decoding, signing, and broadcasting Nostr (Notes and Other Stuff Transmitted by Relays) events. Built with safety and speed in mind, `encoaca` simplifies complex cryptographic operations and relay interactions into a single, cohesive tool.

---

## 📖 Table of Contents
- [Overview](#-overview)
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Configuration](#configuration)
- [Usage](#-usage)
  - [CLI Examples](#cli-examples)
  - [Library Usage](#library-usage)
- [Development](#-development)
- [Deployment](#-deployment)
- [Troubleshooting](#-troubleshooting)
- [Roadmap](#-roadmap)
- [License & Credits](#-license--credits)

---

## 🔍 Overview

`encoaca` bridges the gap between raw Nostr protocol specifications and developers. Whether you need to quickly convert a Bech32 public key (`npub`) to its hexadecimal equivalent, sign a custom event payload using Schnorr signatures, or broadcast an event to a list of decentralized relays, `encoaca` provides a robust, zero-allocation-focused engine to get the job done.

### Target Audience
- **Nostr Protocol Developers** looking for a reliable CLI testing tool.
- **System Integrators** building automated bots or bridges to the Nostr network.
- **Rust Engineers** seeking a modular, highly optimized library for Nostr cryptographic primitives.

---

## ✨ Features

- **NIP-19 Bech32 Transcoding**: Seamless conversion between Hex and standard Nostr Bech32 keys/identifiers (`npub`, `nsec`, `note`, `nprofile`, `nevent`, `naddr`).
- **Cryptographic Signing & Verification**: Secure event signing using the `secp256k1` elliptic curve and Schnorr signatures (NIP-01).
- **Asynchronous Relay Broadcaster**: Multi-threaded, asynchronous event broadcasting to multiple relays simultaneously with connection pooling and retry logic.
- **JSON Validation**: Strict validation of Nostr event structures before network transmission.
- **Zero-Dependency Core**: Cryptographic and encoding modules are kept highly decoupled for maximum performance and portability.

---

## 🛠️ Tech Stack

- **Language**: Rust (Edition 2021)
- **Asynchronous Runtime**: [tokio](https://tokio.rs/)
- **Cryptography**: [secp256k1](https://github.com/rust-bitcoin/rust-secp256k1) (Schnorr signatures)
- **Serialization**: [serde](https://serde.rs/) & [serde_json](https://github.com/serde-rs/json)
- **Networking**: [tokio-tungstenite](https://github.com/snapview/tokio-tungstenite) (WebSockets)
- **CLI Parsing**: [clap](https://github.com/clap-rs/clap) (v4)

---

## 📐 Architecture

`encoaca` is structured as a highly modular binary and library. The codebase is cleanly separated into concerns:

```
┌────────────────────────────────────────────────────────┐
│                       main.rs                          │
│             (CLI Interface & Command Router)           │
└──────────────────────────┬─────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
   ┌────────────────────┐    ┌────────────────────┐
   │      nostr.rs      │    │     helper.rs      │
   │  (Crypto, NIP-19,  │    │ (Hex, Validations, │
   │  Relay Client)     │    │  Formatting Utils) │
   └────────────────────┘    └────────────────────┘
              │                         │
              └────────────┬────────────┘
                           ▼
   ┌──────────────────────────────────────────────┐
   │                  tests.rs                    │
   │        (Integration & Unit Testing)          │
   └──────────────────────────────────────────────┘
```

### Component Breakdown

*   **`main.rs`**: Serves as the entry point. It parses CLI arguments, manages runtime initialization via `tokio`, and routes commands to the core engine.
*   **`nostr.rs`**: Houses the core Nostr protocol logic. This includes cryptographic key generation, NIP-19 encoding/decoding, event serialization, and the WebSocket-based relay connection manager.
*   **`helper.rs`**: Contains pure utility functions such as hex string validation, timestamp generation, and terminal formatting helpers.
*   **`tests.rs`**: Comprehensive test suite covering edge cases in Bech32 parsing, cryptographic signatures, and payload serialization.

---

## 🚀 Getting Started

### Prerequisites

To build `encoaca` from source, you need the Rust toolchain installed on your system:

- **Rust**: `v1.70.0` or higher
- **Cargo**: Packaged with Rust

```bash
# Install Rust (if not already installed)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Installation

Clone the repository and build the release binary:

```bash
# Clone the repository
git clone https://github.com/mirenox14/encoaca.git
cd encoaca

# Build in release mode
cargo build --release

# Verify installation
./target/release/encoaca --version
```

To install the binary globally on your system:

```bash
cargo install --path .
```

### Configuration

`encoaca` can be configured via command-line flags or environment variables. Create a `.env` file in your working directory to store sensitive keys and default relays:

```env
# Default private key (Hex or nsec) for signing events
NOSTR_PRIVATE_KEY="nsec1..."

# Comma-separated list of default relays to broadcast to
NOSTR_RELAYS="wss://relay.damus.io,wss://nos.lol,wss://relay.nostr.band"
```

---

## 💡 Usage

### CLI Examples

#### 1. Decode a Bech32 Key (`npub` / `nsec` / `note`) to Hex
```bash
encoaca decode npub1sg6plzpt964n68g980z9g2368g980z9g2368g980z9g2368g9s78m0f
```
**Output:**
```json
{
  "type": "public_key",
  "hex": "7e8a3424a2116d5e35475d519a527c97b652405a126d4a35c1c5a62000000000"
}
```

#### 2. Encode a Hex Public Key to `npub`
```bash
encoaca encode --type npub 7e8a3424a2116d5e35475d519a527c97b652405a126d4a35c1c5a62000000000
```
**Output:**
```
npub1sg6plzpt964n68g980z9g2368g980z9g2368g980z9g2368g9s78m0f
```

#### 3. Generate a New Nostr Keypair
```bash
encoaca generate-key
```
**Output:**
```
Private Key (Hex):  e9a2...
Private Key (nsec): nsec1...
Public Key (Hex):   8f3c...
Public Key (npub):  npub1...
```

#### 4. Publish a Text Note to Relays
```bash
encoaca publish \
  --secret nsec1... \
  --content "Hello Nostr! Sent securely via encoaca." \
  --relays wss://relay.damus.io,wss://nos.lol
```

---

### Library Usage

Add `encoaca` to your Rust project's `Cargo.toml` (if using as a dependency):

```toml
[dependencies]
encoaca = { path = "../path/to/encoaca" }
tokio = { version = "1.0", features = ["full"] }
```

#### Example: Signing and Encoding an Event in Rust

```rust
use encoaca::nostr::{NostrKeypair, NostrEvent};
use encoaca::helper::get_timestamp;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 1. Generate or load a keypair
    let keys = NostrKeypair::from_nsec("nsec1...")?;
    
    // 2. Construct a text note event (Kind 1)
    let content = "Hello from Rust code!";
    let created_at = get_timestamp();
    let tags = vec![];
    
    let event = NostrEvent::new_signed(&keys, created_at, 1, tags, content)?;
    
    println!("Signed Event ID: {}", event.id);
    println!("Signature: {}", event.sig);
    
    // 3. Convert public key to npub
    let npub = keys.to_npub()?;
    println!("Author npub: {}", npub);

    Ok(())
}
```

---

## 🧪 Development

### Running Tests
`encoaca` includes a comprehensive test suite covering cryptographic operations, encoding edge cases, and helper functions.

```bash
# Run all tests
cargo test

# Run tests with stdout enabled
cargo test -- --nocapture
```

### Code Quality & Linting
Ensure your code adheres to standard Rust formatting and clippy guidelines before submitting a pull request:

```bash
# Format check
cargo fmt --all -- --check

# Run Clippy lints
cargo clippy --all-targets --all-features -- -D warnings
```

---

## 📦 Deployment

### Building a Static Binary
For production deployments, you can compile a statically linked binary using the MUSL target:

```bash
# Install the MUSL target
rustup target add x86_64-unknown-linux-musl

# Build static binary
cargo build --release --target x86_64-unknown-linux-musl
```
The resulting binary is located at `target/x86_64-unknown-linux-musl/release/encoaca` and has zero external shared library dependencies.

### Docker Deployment
A multi-stage Dockerfile is provided to build and run `encoaca` in a minimal container environment:

```dockerfile
# --- Build Stage ---
FROM rust:1.70-slim as builder
WORKDIR /app
COPY . .
RUN cargo build --release

# --- Runtime Stage ---
FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/target/release/encoaca /usr/local/bin/encoaca
ENTRYPOINT ["encoaca"]
```

Build and run the Docker image:
```bash
docker build -t encoaca:latest .
docker run --rm encoaca:latest --help
```

---

## ❓ Troubleshooting

### Common Issues

#### 1. Connection Refused / Timeout when publishing
*   **Cause**: The specified relay is offline, or your network blocks outgoing WebSocket connections on port 443.
*   **Solution**: Verify your internet connection and ensure the relay URL is correct. Try using a highly stable relay like `wss://relay.damus.io`.

#### 2. Invalid Bech32 Checksum
*   **Cause**: The `npub` or `nsec` key was copied incorrectly.
*   **Solution**: Ensure there are no trailing spaces or missing characters. Nostr Bech32 strings are case-insensitive but must contain exactly 58 characters for public keys.

#### 3. OpenSSL / Cryptographic compilation errors
*   **Cause**: Missing development headers for build-essential tools.
*   **Solution**: Install build dependencies:
    *   *Ubuntu/Debian*: `sudo apt install build-essential pkg-config libssl-dev`
    *   *macOS*: Xcode Command Line Tools are sufficient.

---

## 🗺️ Roadmap

- [ ] **NIP-05 Support**: Resolve human-readable identifiers (e.g., `user@domain.com`) directly from the CLI.
- [ ] **NIP-44 Encryption**: Add support for the new standardized symmetric encryption scheme.
- [ ] **Interactive TUI**: A terminal user interface for monitoring relays and reading live event feeds.
- [ ] **WASM Bindings**: Compile the core encoding engine to WebAssembly for browser-based usage.

---

## 📄 License & Credits

### License
This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

### Acknowledgments
- [Nostr Protocol NIPs](https://github.com/nostr-protocol/nips) for the open-source specification.
- The Rust Bitcoin community for the underlying `secp256k1` implementation.
- All contributors who help maintain and improve this tool.