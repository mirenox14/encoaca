
# envo (encoaca)

[![crates.io](https://img.shields.io/badge/crates.io-v0.1.0-orange.svg)](https://crates.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust](https://img.shields.io/badge/rust-1.75%2B-blue.svg)](https://www.rust-lang.org)
[![Built with Nostr](https://img.shields.io/badge/Built%20with-Nostr-purple.svg)](https://nostr.com)

`envo` (repository: `encoaca`) is a secure, decentralized utility designed to synchronize and manage environment configurations (`.env` files) across teams and machines using the **Nostr** protocol. 

By leveraging Nostr's censorship-resistant relay network and robust cryptographic primitives, `envo` provides end-to-end encrypted (E2EE) storage and transport for your project's secrets without relying on centralized third-party secret managers like HashiCorp Vault, AWS Secrets Manager, or 1Password.

--

## 📖 Table of Contents

1. [Overview](#-overview)
2. [Features](#-features)
3. [How It Works (Architecture)](#-how-it-works-architecture)
4. [Tech Stack](#-tech-stack)
5. [Getting Started](#-getting-started)
   - [Prerequisites](#prerequisites)
   - [Installation](#installation)
   - [Configuration](#configuration)
6. [Usage Guide](#-usage-guide)
   - [1. Generate Identity Keys](#1-generate-identity-keys)
   - [2. Push Environment Variables](#2-push-environment-variables)
   - [3. Pull Environment Variables](#3-pull-environment-variables)
7. [Security Model](#-security-model)
8. [Development](#-development)
9. [Troubleshooting](#-troubleshooting)
10. [License](#-license)

---

## 🔍 Overview

Modern development workflows require sharing environment configurations (`.env` files) across multiple developer machines and CI/CD pipelines. Traditional methods (sharing via Slack, committing encrypted files to Git, or using heavy enterprise secret managers) are either insecure, tedious, or expensive.

`envo` solves this by treating **Nostr relays** as a decentralized, highly available backend. 
- **Encryption-First**: All environment payloads are encrypted locally before leaving your machine.
- **Cryptographic Identity**: Updates are signed using Nostr private keys (`nsec`), ensuring absolute authenticity.
- **Trust-On-First-Use (TOFU)**: Securely track trusted publishers (`npub`) for specific configuration tags.

---

## ✨ Features

*   **Decentralized Sync**: No central database or SaaS subscription required. Uses standard Nostr relays.
*   **End-to-End Encryption (E2EE)**: Payloads are encrypted using industry-standard cryptographic algorithms before transmission.
*   **Cryptographic Signatures**: Every configuration state is signed, preventing tampering and unauthorized overrides.
*   **Trust-On-First-Use (TOFU) Verification**: Bind specific configuration tags to trusted publisher public keys (`npub`).
*   **Simple CLI Interface**: Intuitive commands (`keygen`, `push`, `pull`) designed to fit seamlessly into developer workflows and CI/CD pipelines.
*   **Zero-Noise Error Handling**: Clean, developer-friendly terminal output that integrates perfectly with Unix pipes and automation scripts.

---

## 🏗️ How It Works (Architecture)

```
                       ┌────────────────────────┐
                       │   Local Developer PC   │
                       └───────────┬────────────┘
                                   │
                    1. Encrypts .env with Nostr Keys
                    2. Signs payload with nsec
                                   │
                                   ▼
                       ┌────────────────────────┐
                       │   Nostr Relay Network  │
                       │  (Decentralized Nodes) │
                       └───────────┬────────────┘
                                   │
                    3. Fetches encrypted event
                    4. Verifies signature & npub trust
                                   │
                                   ▼
                       ┌────────────────────────┐
                       │     Target Machine     │
                       │   (CI/CD or Colleague) │
                       └────────────────────────┘
```

### Directory Structure

The codebase is modularly designed for performance, security, and maintainability:

```
├── main.rs                 # CLI entry point, command routing, and error handling
├── commands.rs             # Subcommand module declarations
│   ├── key_gen.rs          # Nostr keypair generation utility
│   ├── push.rs             # Local encryption and relay publishing logic
│   └── pull.rs             # Remote fetching, trust verification, and decryption logic
├── nostr.rs                # Core Nostr protocol operations
│   ├── build_and_sign_event.rs # Event creation and cryptographic signing
│   ├── encrypter.rs        # Payload encryption/decryption routines
│   ├── fetch_event.rs      # Querying relays for tagged events
│   └── publish_event.rs    # Broadcasting signed events to relays
├── helper.rs               # Utility modules
│   ├── env_files.rs        # Parsing and loading local .env files
│   ├── event_content.rs    # Structuring event payloads
│   ├── key_valid.rs        # Validation for npub/nsec keys
│   ├── log.rs              # Structured terminal logging
│   ├── relay_provider.rs   # Relay connection pool management
│   ├── secret_file.rs      # Secure local storage of the user's private key
│   └── trusted_owners.rs   # Local database of trusted tag-to-npub mappings
└── tests.rs                # Integration and unit tests
```

---

## 💻 Tech Stack

*   **Language**: [Rust](https://www.rust-lang.org/) (Edition 2021)
*   **Async Runtime**: [Tokio](https://tokio.rs/)
*   **CLI Parser**: [Clap](https://github.com/clap-rs/clap) (v4 with derive features)
*   **Protocol**: [Nostr](https://github.com/nostr-protocol/nips) (utilizing cryptographic event signatures and relay communication)
*   **Cryptography**: End-to-end encryption utilizing Nostr-native key derivation (NIP-04/NIP-44 compliant structures)

---

## 🚀 Getting Started

### Prerequisites

*   [Rust toolchain](https://rustup.rs/) (v1.75 or higher recommended)
*   Access to internet-enabled Nostr relays (default public relays are supported out-of-the-box)

### Installation

#### From Source

1. Clone the repository:
   ```bash
   git clone https://github.com/mirenox14/encoaca.git
   cd encoaca
   ```

2. Build the release binary:
   ```bash
   cargo build --release
   ```

3. Move the binary to your system path:
   ```bash
   mv target/release/envo /usr/local/bin/
   ```

### Configuration

`envo` looks for a local configuration directory to store your private identity key and the list of trusted publishers.

*   **Global Secrets Path**: `~/.config/envo/secret.json` (stores your generated `nsec`)
*   **Trusted Publishers Database**: `~/.config/envo/trusted_owners.json` (stores mappings of `tag -> npub`)

---

## 📖 Usage Guide

### 1. Generate Identity Keys

Before pushing or pulling configurations, generate your unique cryptographic identity.

bash
encoaca keygen

*This generates a new Nostr keypair, securely saving your private key (`nsec`) locally and printing your public key (`npub`) to the console.*

### 2. Push Environment Variables

To encrypt and upload your local environment variables to the relay network under a specific tag:

bash
encoaca push my-project-production


**What happens under the hood:**
1. `encoaca` scans the current directory for environment files (e.g., `.env`).
2. The contents are serialized and encrypted using your Nostr keys.
3. A signed Nostr event is constructed containing the encrypted payload, tagged with `my-project-production`.
4. The event is broadcasted to configured Nostr relays.

### 3. Pull Environment Variables

To fetch, verify, and decrypt environment variables published under a tag:

#### First-Time Pull (Trust-On-First-Use)
When pulling a tag for the first time, you must explicitly specify the publisher's public key (`npub`) to establish trust:

bash
encoaca pull my-project-production --owner npub1h8...39ax


#### Subsequent Pulls
Once trust is established, you no longer need to pass the `--owner` flag. `encoaca` remembers the trusted publisher for that tag:

bash
encoaca pull my-project-production


**What happens under the hood:**
1. `encoaca` queries Nostr relays for events matching the tag `my-project-production`.
2. It filters events to ensure they are signed by the trusted owner's `npub`.
3. The encrypted payload is downloaded, decrypted locally using your keys, and written back to your local environment file.

---
## 🔒 Security Model

`envo` is built with a strict security-first mindset:

1.  **Zero Plaintext Leakage**: Your raw environment variables are never transmitted over the network or stored on relays in plaintext.
2.  **Asymmetric Encryption**: Payloads are encrypted using ephemeral shared secrets derived via Diffie-Hellman key exchange between your identity key and the target recipient's public key.
3.  **Cryptographic Integrity**: Nostr events are immutable once published; any tampering with the payload invalidates the signature, causing `envo pull` to reject the update.
4.  **Replay Attack Protection**: Events contain timestamps and unique identifiers, preventing malicious actors from rolling back your configuration to an older, potentially vulnerable state.

---

## 🛠️ Development

### Running Tests

To run the test suite and verify cryptographic operations:

bash
cargo test


### Code Style & Guidelines

This project adheres to standard Rust formatting guidelines. Ensure your code is formatted before submitting a pull request:

bash
cargo fmt --all
cargo clippy --all-targets --all-features -- -D warnings


---
## ❓ Troubleshooting

### Common Issues

#### 1. "Relay connection failed"
*   **Cause**: The default public Nostr relays might be offline or rate-limiting your IP.
*   **Solution**: Ensure your internet connection is stable. Future versions will support custom relay configurations via an `envo.toml` file.

#### 2. "Untrusted publisher signature"
*   **Cause**: The event found on the relay for the specified tag was signed by a key different from the one saved in your trusted owners database.
*   **Solution**: If the publisher has intentionally rotated their key, update your trust configuration or run the pull command with the new `--owner <npub>` flag to override.

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

### 🌟 Acknowledgments

*   The [Nostr Protocol](https://github.com/nostr-protocol/nostr) community for creating a robust, decentralized communication standard.
*   The Rust community for providing world-class cryptographic and asynchronous tooling.
