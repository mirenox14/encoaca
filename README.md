
# encoaca

`encoaca` is a Rust-based command-line tool designed to interact with the Nostr protocol. It provides utilities for Nostr key generation, publishing events to Nostr relays, and fetching data from the Nostr network.

## Features

- **Key Generation**: Generate cryptographic key pairs compatible with Nostr.
- **Push Operations**: Publish events and data to Nostr relays.
- **Pull Operations**: Retrieve and fetch events from Nostr relays.
- **Cross-Platform Setup**: Automated installation scripts for Linux, macOS, and Windows.

## Installation

### Quick Install

#### Linux / macOS

Execute the shell installer script:

```bash
chmod +x install.sh
./install.sh
```

#### Windows (PowerShell)

Execute the PowerShell installation script:

```powershell
.\install.ps1
```

### Building from Source

Ensure you have [Rust and Cargo](https://www.rust-lang.org/tools/install) installed.

```bash
git clone https://github.com/mirenox14/encoaca.git
cd encoaca
cargo build --release
```

The compiled binary will be available in the `target/release/` directory.

## Usage

`encoaca` provides commands for Nostr identity management and relay operations.

### Key Generation

Generate a new Nostr key pair:

```bash
cargo run -- key-gen
```

### Push to Nostr Relays

Publish events or data to configured relays:

```bash
cargo run -- push
```

### Pull from Nostr Relays

Fetch events or data from specified relays:

```bash
cargo run -- pull
```

## Testing

Run the test suite using Cargo:

```bash
cargo test
```

## Project Structure

```
├── main.rs            # CLI entry point and argument parsing
├── nostr.rs           # Nostr protocol implementation and relay handler
├── key_gen.rs         # Key pair generation module
├── key_valid.rs       # Key validation and identity directory management
├── push.rs            # Logic for publishing data to Nostr relays
├── pull.rs            # Logic for retrieving data from Nostr relays
├── env_files.rs       # Environment file parsing and recipient key loader
├── event_content.rs   # Event payload and recipient mappings
├── helper.rs          # Common utilities and helper functions
├── log.rs             # Terminal logging and status output
├── relay_provider.rs # Default Nostr relay provider
├── secret_file.rs     # File permission enforcement and secret file storage
├── trusted_owners.rs  # Trusted owner pinning for pull validation
├── tests.rs           # Unit and integration tests
├── install.sh         # Shell installer for Unix-like systems
└── install.ps1        # PowerShell installer for Windows
```