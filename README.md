# encoaca

`encoaca` is a command-line tool written in Rust designed for securely managing, encrypting, and synchronizing `.env` files and secret files across environments using the Nostr protocol.

By leveraging Nostr relays, encrypted event contents, key pair validation, and trusted owner access controls, `encoaca` provides a decentralized mechanism to share environment secrets safely between trusted parties or machines.

---

## Features

- **Nostr Relay Integration:** Publish and retrieve encrypted secrets across configurable Nostr relays.
- **Environment & Secret File Management:** Easily push and pull `.env` configurations and sensitive files across platforms.
- **Key Generation & Validation:** Generate cryptographic keys and validate key signatures to ensure secure access control.
- **Trusted Owners Access Control:** Restrict secret access and event processing to configured, trusted identity keys.
- **Cross-Platform Installer:** Easy installation options provided for macOS/Linux (`install.sh`) and Windows (`install.ps1`).

---

## Installation

### Automatic Installation

#### Linux / macOS
Run the shell installation script:
```bash
curl -sSL https://raw.githubusercontent.com/mirenox14/encoaca/main/install.sh | bash
```

#### Windows (PowerShell)
Run the PowerShell installation script:
```powershell
iwr -useb https://raw.githubusercontent.com/mirenox14/encoaca/main/install.ps1 | iex
```

---

### Building from Source

Ensure you have Rust and Cargo installed. Clone the repository and build the project:

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
cargo build --release
```

The compiled binary will be available at `target/release/encoaca`.

---

## Core Architecture & Module Breakdown

- **`main.rs`**: Entry point for command parsing and execution flow.
- **`push.rs`**: Encrypts and publishes `.env` and secret files to designated Nostr relays.
- **`pull.rs`**: Fetches, authenticates, and decrypts secret payloads from Nostr relays.
- **`key_gen.rs` & `key_valid.rs`**: Handles identity key pair generation, formatting, and validity checking.
- **`trusted_owners.rs`**: Manages allowed keys/identities authorized to interact with published secret payloads.
- **`relay_provider.rs` & `nostr.rs`**: Encapsulates connectivity, event construction, and messaging over Nostr relays.
- **`env_files.rs` & `secret_file.rs`**: File I/O helpers for scanning, processing, and outputting local environment and secret files.
- **`event_content.rs`**: Defines Nostr event content structure and encryption format.
- **`log.rs` & `helper.rs`**: Provides application-wide logging formats and utility helpers.

---

## Basic Workflow

1. **Generate Identity Keys:** Generate your key pair using the key generation commands.
2. **Configure Relays & Trusted Owners:** Set up target Nostr relays and add public keys of trusted owners allowed to sync secrets.
3. **Push Secrets:** Encrypt and push local `.env` or secret files to the relay network.
4. **Pull Secrets:** Authenticate and pull the latest secret files on target environments.

---

## Development & Testing

To run the test suite:

```bash
cargo test
```

---

## License

This project is maintained by [mirenox14](https://github.com/mirenox14). Contributions and issue reports are welcome.