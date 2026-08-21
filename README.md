# envo

[![crates.io](https://img.shields.io/badge/crates.io-v0.1.0-orange.svg)](https://crates.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust](https://img.shields.io/badge/rust-1.75%2B-blue.svg)](https://www.rust-lang.org)
[![Built with Nostr](https://img.shields.io/badge/Built%20with-Nostr-purple.svg)](https://nostr.com)

`envo` is a decentralized environment configuration manager that uses the Nostr protocol for secure, encrypted synchronization of `.env` files. It eliminates centralized secret managers while ensuring cryptographic verification and end-to-end encryption.

---

## ✨ Features

- **Decentralized Sync**  
  Uses Nostr relays for distributed storage and retrieval of encrypted configuration data

- **End-to-End Encryption**  
  Leverages NIP-04/NIP-44 for local encryption before transmission

- **Trust Management**  
  Supports Trust-On-First-Use (TOFU) and explicit key verification via Nostr public keys (`npub`)

- **Minimalist CLI**  
  Simple, Unix-style commands for developers and automation

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

**Core Components**:
- CLI interface for command routing and error handling
- Nostr integration for event signing, encryption, and relay communication

---

## 🛠️ Installation

```bash
git clone https://github.com/mirenox14/envo.git
cd envo
cargo build --release
mv target/release/envo /usr/local/bin/
```

**Requirements**:
- Rust 1.75+
- Nostr relay access

---

## 📚 Usage

### Key Management
```bash
envo keygen      # Generate Nostr keypair (stores nsec in ~/.config/envo/secret.json)
```

### Configuration Sync

**Push**:
```bash
envo push <tag>  # Encrypts .env and publishes signed event with specified tag
```

**Pull**:
```bash
# Initial trust setup
envo pull <tag> --owner <npub>

# With trusted publisher
envo pull <tag>  # Verifies against ~/.config/envo/trusted_owners.json
```

---

## 🔐 Security Model

- Secrets remain encrypted during transmission and storage
- Events include timestamps and unique IDs for replay protection
- Decryption only occurs on the target machine
- Trust enforced through verified `npub` keys

---

## 🧪 Development

```bash
cargo test    # Run test suite
cargo fmt     # Format code
cargo clippy  # Lint checks
```

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file

---

## 🌟 Acknowledgments

- [Nostr Protocol](https://github.com/nostr-protocol/nostr) community
- Rust ecosystem for cryptographic and async tooling