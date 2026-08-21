# envo

[![crates.io](https://img.shields.io/badge/crates.io-v0.1.0-orange.svg)](https://crates.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust](https://img.shields.io/badge/rust-1.75%2B-blue.svg)](https://www.rust-lang.org)
[![Built with Nostr](https://img.shields.io/badge/Built%20with-Nostr-purple.svg)](https://nostr.com)

`envo` is a secure, decentralized environment configuration manager using the **Nostr protocol** to synchronize and manage `.env` files across teams and machines. It provides end-to-end encryption and cryptographic identity verification without centralized secret managers.

---

## 📚 Overview

Modern workflows require secure environment configuration sharing across machines and CI/CD pipelines. Traditional methods (Slack sharing, encrypted Git, or enterprise tools) are either insecure, tedious, or expensive.

`envo` solves this by using Nostr relays as a decentralized backend:
- **Encryption-First**: Payloads are encrypted locally before transmission
- **Cryptographic Identity**: Updates are signed with Nostr keys (`nsec`)
- **Trust Model**: Uses Trust-On-First-Use (TOFU) for publisher verification

---

## ✨ Features

- **Decentralized Sync**  
  No central database required - uses standard Nostr relays

- **End-to-End Encryption (E2EE)**  
  Industry-standard encryption before transmission

- **Cryptographic Signatures**  
  All updates are signed to prevent tampering

- **Trust Verification**  
  Bind configuration tags to trusted publisher keys (`npub`)

- **Simple CLI**  
  Intuitive commands for developers and CI/CD pipelines

- **Zero-Noise Output**  
  Clean terminal output compatible with Unix pipes

---

## 🧠 Architecture

```
Local Machine
 └── Encrypts .env with Nostr keys
 └── Signs payload with nsec
 └── Publishes to Nostr relays

Target Machine
 └── Fetches encrypted event
 └── Verifies signature & npub trust
 └── Decrypts and writes .env
```

Key components:
- **CLI**: Command routing and error handling
- **Nostr Integration**: Event signing, encryption, relay communication
- **Trust Store**: Local database of trusted `tag -> npub` mappings

---

## 🛠️ Tech Stack

- **Language**: Rust (Edition 2021)
- **Async Runtime**: Tokio
- **CLI Parser**: Clap v4
- **Protocol**: Nostr (NIP-04/NIP-44 compliant)
- **Cryptography**: Rust-native cryptographic primitives

---

## 🚀 Getting Started

### Prerequisites

- Rust toolchain (v1.75+)
- Internet access to Nostr relays

### Installation

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
cargo build --release
mv target/release/envo /usr/local/bin/
```

### Configuration Files

- **Private Key**: `~/.config/envo/secret.json` (stores `nsec`)
- **Trusted Publishers**: `~/.config/envo/trusted_owners.json`

---

## 📖 Usage

### 1. Generate Identity Keys

```bash
envo keygen
```

Generates a Nostr keypair and stores the private key locally.

### 2. Push Configuration

```bash
envo push my-project-production
```

1. Scans for `.env` files
2. Encrypts with your Nostr keys
3. Publishes signed event with tag `my-project-production`

### 3. Pull Configuration

**First-time use** (establish trust):
```bash
envo pull my-project-production --owner npub1h8...39ax
```

**Subsequent pulls** (trusted publisher):
```bash
envo pull my-project-production
```

1. Queries relays for events with matching tag
2. Verifies signature against trusted `npub`
3. Decrypts and writes to local `.env`

---

## 🔒 Security Model

- **Zero Plaintext Exposure**: No secrets transmitted in plaintext
- **Immutable Events**: Nostr signatures prevent tampering
- **Replay Protection**: Events include timestamps and unique IDs
- **Local Decryption**: Secrets are only decrypted on target machine

---

## 🧪 Development

```bash
cargo test          # Run test suite
cargo fmt --all     # Format code
cargo clippy        # Lint checks
```

---

## ❓ Troubleshooting

### Relay Connection Issues

- **Cause**: Default relays may be offline or rate-limiting
- **Solution**: Future versions will support custom relay configuration

### Untrusted Publisher Signature

- **Cause**: Event signed by unexpected key
- **Solution**: Use `--owner <npub>` to override trust or update trust store

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file

---

## 🌟 Acknowledgments

- [Nostr Protocol](https://github.com/nostr-protocol/nostr) community
- Rust ecosystem for cryptographic and async tooling