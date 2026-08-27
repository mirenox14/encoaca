# encoaca

`encoaca` is a command-line tool written in Rust for securely managing, encrypting, pushing, and pulling environment variables and secret files using the Nostr network.

## Features

- **Nostr Integration**: Store and sync encrypted secrets and environment configurations over decentralized Nostr relays.
- **Push & Pull Commands**: Easily upload local `.env` and secret files to relays or retrieve them when needed.
- **Key Generation & Validation**: Utilities to generate and validate cryptographic keys used for authentication and encryption.
- **Trusted Owners**: Access controls to restrict pull and push permissions to authorized Nostr public keys.
- **Relay Provider Support**: Configurable Nostr relay connections for broadcasting and fetching encrypted secret events.

## Installation

### Unix / macOS (Shell Script)

```bash
curl -fsSL https://raw.githubusercontent.com/mirenox14/encoaca/main/install.sh | bash
```

### Windows (PowerShell)

```powershell
iwr -useb https://raw.githubusercontent.com/mirenox14/encoaca/main/install.ps1 | iex
```

### Building from Source

Requires Rust and `cargo` installed.

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
cargo build --release
```

The compiled binary will be located in `target/release/encoaca`.

## Usage

### Build and Run

Run `encoaca` directly using `cargo`:

```bash
cargo run -- [COMMAND] [OPTIONS]
```

Or execute the built binary:

```bash
./target/release/encoaca [COMMAND] [OPTIONS]
```

### Key Management

Generate cryptographic keys for signing and decrypting Nostr events:

```bash
cargo run -- key-gen
```

### Push Secrets

Encrypt and publish environment or secret files to connected Nostr relays:

```bash
cargo run -- push
```

### Pull Secrets

Fetch and decrypt environment or secret files from Nostr relays:

```bash
cargo run -- pull
```

## Project Structure

- `main.rs` - CLI entry point and command router.
- `push.rs` - Logical handling for encrypting and publishing secret events to relays.
- `pull.rs` - Handling for fetching and decrypting secret events from relays.
- `env_files.rs` - Reading and parsing environment variables and `.env` files.
- `secret_file.rs` - File handling and encapsulation for secret payloads.
- `event_content.rs` - Formatting and constructing Nostr event payloads.
- `nostr.rs` - Core Nostr protocol operations and event handling.
- `relay_provider.rs` - Management of relay connection pools and provider communications.
- `key_gen.rs` & `key_valid.rs` - Keypair generation and validation functions.
- `trusted_owners.rs` - Public key authorization and identity checks.
- `helper.rs` - Utility functions used across modules.
- `log.rs` - Logging and output formatting.
- `tests.rs` - Test suite for verification.

## Testing

Run the test suite using `cargo`:

```bash
cargo test
```