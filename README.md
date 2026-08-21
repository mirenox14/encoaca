# envo

[![crates.io](https://img.shields.io/badge/crates.io-v0.1.0-orange.svg)](https://crates.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Rust](https://img.shields.io/badge/rust-1.75%2B-blue.svg)](https://www.rust-lang.org)
[![Built with Nostr](https://img.shields.io/badge/Built%20with-Nostr-purple.svg)](https://nostr.com)

`envo` is a decentralized environment configuration manager that uses the Nostr protocol for secure, encrypted synchronization of `.env` files across teams and machines. It eliminates the need for centralized secret managers while ensuring cryptographic verification and end-to-end encryption.

---

## 🔐 Core Features

- **Decentralized Sync**  
  Leverages Nostr relays for distributed storage and retrieval

- **End-to-End Encryption**  
  Secrets are encrypted locally before transmission using NIP-04/NIP-44

- **Cryptographic Trust**  
  All updates are signed with Nostr private keys (`nsec`) and verified against trusted publishers (`npub`)

- **Trust-On-First-Use (TOFU)**  
  Establishes trust through initial manual verification of publisher keys

- **Minimalist CLI**  
  Simple, Unix-friendly commands for developers and automation

---

## 🧱 Architecture

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
- **Nostr Integration**: Event signing, encryption, and relay communication
- **Trust Store**: Local database mapping configuration tags to trusted `npub` keys

---

## 🛠️ Installation

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
cargo build --release
mv target/release/envo /usr/local/bin/
```

**Requirements**:
- Rust toolchain (v1.75+)
- Internet access to Nostr relays

---

## 📖 Usage

### 1. Generate Identity Keys

```bash
envo keygen
```

Creates a Nostr keypair and stores the private key (`nsec`) in `~/.config/envo/secret.json`.

### 2. Push Configuration

```bash
envo push my-project-production
```

- Encrypts `.env` files using your Nostr keys
- Publishes a signed event with the tag `my-project-production`

### 3. Pull Configuration

**Initial trust setup**:
```bash
envo pull my-project-production --owner npub1h8...39ax
```

**Trusted publisher**:
```bash
envo pull my-project-production
```

- Fetches events with the specified tag
- Verifies signatures against trusted `npub` in `~/.config/envo/trusted_owners.json`
- Decrypts and writes `.env` files to the current directory

---

## 🔒 Security Model

- Secrets remain encrypted during transmission and storage
- Events include timestamps and unique IDs for replay protection
- Decryption only occurs on the target machine
- Trust is enforced through verified `npub` keys

---

## 🧪 Development

```bash
cargo test          # Run test suite
cargo fmt --all     # Format code
cargo clippy        # Lint checks
```

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file

---

## 🌟 Acknowledgments

- [Nostr Protocol](https://github.com/nostr-protocol/nostr) community
- Rust ecosystem for cryptographic and async tooling