# 🦜 encoaca

[![Rust](https://img.shields.io/badge/rust-1.75%2B-blue.svg?style=flat-square&logo=rust)](https://www.rust-lang.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg?style=flat-square)]()
[![Nostr Protocol](https://img.shields.io/badge/nostr-protocol-purple.svg?style=flat-square)](https://github.com/nostr-protocol/nips)

`encoaca` is an ultra-fast, secure, and lightweight command-line utility and library written in Rust, designed for encoding, decoding, and validating Nostr protocol data structures. It provides seamless conversions between raw cryptographic formats (hexadecimal) and human-readable Nostr-specific formats (Bech32/NIP-19), alongside key generation and signature verification utilities.

Whether you are a Nostr developer debugging event payloads, a power user managing multiple identities, or an automated script pipeline processing Nostr data, `encoaca` offers a robust, zero-dependency-adjacent CLI interface optimized for speed and correctness.

---

## 📖 Table of Contents
- [Features](#-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Verification](#verification)
- [Configuration](#-configuration)
- [Usage](#-usage)
  - [Key Generation](#1-key-generation)
  - [Encoding (Hex to Bech32/NIP-19)](#2-encoding-hex-to-bech32nip-19)
  - [Decoding (Bech32/NIP-19 to Hex)](#3-decoding-bech32nip-19-to-hex)
  - [Event Verification](#4-event-verification)
- [Development](#-development)
  - [Running Tests](#running-tests)
  - [Code Quality](#code-quality)
- [Deployment](#-deployment)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

---

## ✨ Features

*   **Bi-directional Bech32 (NIP-19) Conversion**: Effortlessly convert keys and entities between raw hex and standard Nostr prefixes:
    *   `npub` (Public Keys) / `nsec` (Private Keys)
    *   `note` (Event IDs)
    *   `nprofile` (Profiles with relay hints)
    *   `nevent` (Events with relay hints)
    *   `naddr` (Parameterized replaceable events)
*   **Cryptographic Utilities**:
    *   Secure, entropy-pure keypair generation (using `secp256k1`).
    *   Derivation of public keys (`npub`/hex) directly from private keys (`nsec`/hex).
    *   Schnorr signature verification for Nostr events.
*   **Developer-Friendly CLI**:
    *   Supports structured JSON output for easy integration with shell scripts (`jq`).
    *   Accepts piped input via standard input (`stdin`).
    *   Colorized, human-readable terminal output.
*   **Memory Safe & Fast**: Built entirely in Rust, ensuring minimal footprint, zero memory leaks, and sub-millisecond execution times.

---

## 🛠️ Tech Stack

*   **Language**: Rust (Edition 2021)
*   **Cryptography**: `secp256k1` (for Schnorr signatures and key operations)
*   **Encoding**: `bech32` (for NIP-19 entity formatting)
*   **Serialization**: `serde` & `serde_json` (for structured CLI outputs)
*   **CLI Parsing**: `clap` (v4, with derive features for robust argument parsing)

---

## 🏗️ Architecture

The project is structured modularly to separate CLI orchestration, core cryptographic/Nostr logic, and helper utilities:

```
├── Cargo.toml            # Project dependencies and metadata
├── src/
│   ├── main.rs           # CLI Entrypoint, argument parsing, and execution routing
│   ├── nostr.rs          # Core Nostr protocol logic, NIP-19 encoding/decoding, cryptography
│   ├── helper.rs         # General-purpose utility functions (hex parsing, formatting, stdin handling)
│   └── tests.rs          # Comprehensive unit and integration test suite
```

### Component Interaction Flow

```
[User / Shell CLI] ──(Input Args/Stdin)──> [main.rs (Clap Parser)]
                                                    │
                                                    ▼
                                          [helper.rs (Sanitization)]
                                                    │
                                                    ▼
                                          [nostr.rs (Crypto / Bech32)]
                                                    │
[Console / JSON Output] <──(Formatted Result)───────┘
```

---

## 🚀 Getting Started

### Prerequisites

To compile `encoaca` from source, you need the Rust toolchain installed on your system:

*   **Rustc**: `v1.75.0` or higher
*   **Cargo**: Package manager for Rust

If you do not have Rust installed, set it up via [rustup](https://rustup.rs/):
```bash
curl --proto '=https' --tlv1.2 -sSf https://sh.rustup.rs | sh
```

### Installation

1.  **Clone the Repository**:
    ```bash
    git clone https://github.com/mirenox14/encoaca.git
    cd encoaca
    ```

2.  **Build the Release Binary**:
    ```bash
    cargo build --release
    ```
    The compiled binary will be available at `./target/release/encoaca`.

3.  **Install Locally**:
    To install the binary directly into your cargo binary path (`~/.cargo/bin`):
    ```bash
    cargo install --path .
    ```

### Verification

Verify that the installation was successful by checking the version:
```bash
encoaca --version
```

---

## ⚙️ Configuration

`encoaca` runs out of the box without any external configuration files. However, you can control its behavior using environment variables:

| Environment Variable | Description | Default Value |
|----------------------|-------------|---------------|
| `ENCOACA_LOG`        | Log level (`error`, `warn`, `info`, `debug`, `trace`) | `warn` |
| `NOSTR_DEFAULT_RELAY`| Default relay hint fallback when encoding profiles | `wss://relay.damus.io` |

---

## 💡 Usage

`encoaca` provides a clean, nested command structure. Run `encoaca help` at any time to see available commands.

### 1. Key Generation
Generate a secure, cryptographically random Nostr keypair.

```bash
# Generate a standard keypair (outputs both hex and Bech32 formats)
encoaca keys generate

# Output keypair in JSON format for scripting
encoaca keys generate --json
```

### 2. Encoding (Hex to Bech32/NIP-19)
Convert raw hexadecimal public or private keys into NIP-19 compliant formats.

```bash
# Encode a hex public key to npub
encoaca encode npub 3bf0c63fc03b0fc81440ec0feb0c44ef98b7a4e4a25c5700e2b4e9802be649fb

# Encode a hex private key to nsec
encoaca encode nsec e9802be649fb3bf0c63fc03b0fc81440ec0feb0c44ef98b7a4e4a25c5700e2b4

# Encode an event ID to note format
encoaca encode note a5d80a1e05030b56b26d36e29783f8a032223a5099b26d36e29783f8a032223a
```

### 3. Decoding (Bech32/NIP-19 to Hex)
Decode any NIP-19 string back into its hexadecimal representation.

```bash
# Decode an npub
encoaca decode npub180cvv07jd69v0as6f0667699m0f6ar33658zcl6v74gnsm6v00qsm6v00q

# Decode an nsec
encoaca decode nsec1vl6v74gnsm6v00qsm6v00qsm6v00qsm6v00qsm6v00qsm6v00qsq6v74g

# Decode using auto-detection (automatically detects prefix)
encoaca decode auto npub180cvv07jd69v0as6f0667699m0f6ar33658zcl6v74gnsm6v00qsm6v00q
```

### 4. Event Verification
Validate the signature and integrity of a Nostr event payload.

```bash
# Verify an event signature by passing a JSON payload
encoaca verify event '{
  "id": "4376c656f143c3d0e266ee8d64290182fd685749c0c44ef98b7a4e4a25c5700e",
  "pubkey": "3bf0c63fc03b0fc81440ec0feb0c44ef98b7a4e4a25c5700e2b4e9802be649fb",
  "created_at": 1672531199,
  "kind": 1,
  "tags": [],
  "content": "Hello, Nostr!",
  "sig": "908a153c535b1f2b576b26d36e29783f8a032223a50d9b26d36e29783f8a032223a50d9b26d36e29783f8a032223a50d9b26d36e29783f8a032223a50d9b26d"
}'
```

---

## 🛠️ Development

### Running Tests
The project features a comprehensive test suite in `src/tests.rs` covering edge cases, invalid checksums, key derivation, and signature verifications.

To run the test suite:
```bash
cargo test
```

To run tests with console output enabled:
```bash
cargo test -- --nocapture
```

### Code Quality
We enforce strict code quality standards. Before submitting a Pull Request, please ensure your code passes formatting and linting checks:

```bash
# Format check
cargo fmt --all -- --check

# Linter check
cargo clippy --all-targets --all-features -- -D warnings
```

---

## 📦 Deployment

To compile a highly optimized, statically linked binary for production deployment:

```bash
# Production build with Link-Time Optimization (LTO)
RUSTFLAGS="-C target-cpu=native" cargo build --release
```

### Cross-Compilation (Linux to Windows/macOS)
You can cross-compile `encoaca` using `cross`:

```bash
# Install cross
cargo install cross --git https://github.com/cross-rs/cross

# Build for Windows
cross build --target x86_64-pc-windows-gnu --release

# Build for Linux ARM64
cross build --target aarch64-unknown-linux-gnu --release
```

---

## ❓ Troubleshooting

### Common Issues

#### 1. Error: `Invalid checksum` during decoding
*   **Cause**: The Bech32 string was copied incorrectly, or characters were mutated (e.g., case-sensitivity issues or missing trailing characters).
*   **Solution**: Double-check the exact string length and characters. Note that Bech32 excludes the characters `1`, `b`, `i`, and `o` to prevent visual confusion.

#### 2. Error: `Invalid public key length`
*   **Cause**: Nostr uses 32-byte (64 character) Schnorr public keys, but a 33-byte (66 character) ECDSA public key was provided.
*   **Solution**: Ensure you are using the x-only coordinate of the public key (32 bytes).

---

## 🤝 Contributing

Contributions are welcome! If you'd like to improve `encoaca`, please follow these steps:

1.  **Fork** the repository.
2.  **Create a feature branch** (`git checkout -b feature/amazing-feature`).
3.  **Commit your changes** (`git commit -m 'Add some amazing feature'`).
4.  **Push to the branch** (`git push origin feature/amazing-feature`).
5.  **Open a Pull Request**.

Please ensure your commits follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

---

## 📄 License

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for details.

---

### Acknowledgments
*   The [Nostr Protocol NIPs](https://github.com/nostr-protocol/nips) community for establishing standard specifications.
*   The authors of the `bech32` and `secp256k1` Rust crates.
