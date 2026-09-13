# envo

`envo` is a command-line tool for sharing encrypted environment files across teams using Nostr relays and NIP-44 asymmetric encryption.

## Features

- **Nostr Identity Management**: Generates and manages local Nostr keypairs (`npub`/`nsec`) stored in `~/.envo/keys.json`.
- **Encrypted Env Sharing**: Encrypts local `.env` content individually for each recipient listed in `.env-share`, plus the publisher.
- **Relay-Based Distribution**: Publishes encrypted payloads to public Nostr relays without requiring a centralized server.
- **Pinned Tag Ownership**: Pin owner public keys per tag in `~/.envo/trusted_owners.json` on first pull to prevent spoofed environment updates.
- **File Security**: Enforces strict filesystem permissions (`0600` for secret files, `0700` for `~/.envo` on Unix platforms).

## Installation

### Shell Script (Linux / macOS)

Run the installation script to download prebuilt binaries to `$HOME/.local/bin`:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

You can customize the installation using environment variables:

- `ENVO_VERSION`: The release tag to install (defaults to `latest`).
- `ENVO_INSTALL_DIR`: The directory where the binary is placed (defaults to `$HOME/.local/bin`).

### From Source

Build and install directly from the source repository using Cargo:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

`envo` relies on two files in your current working directory when pushing secrets:

- `.env`: Contains the environment variables to be shared.
- `.env-share`: A comma-separated list of recipient Nostr public keys (`npub...` or hex) allowed to decrypt the secrets.

Local state and keys are maintained in your home directory:

- `~/.envo/keys.json`: Stores your generated Nostr identity (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Records pinned owner public keys for each tag pulled.

### Relays

`envo` publishes and fetches events using the following Nostr relays:
- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`

## Usage

### Generate an Identity

Create a new Nostr keypair or display an existing identity:

```sh
envo keygen
```

### Push Environment Secrets

Encrypt and publish the local `.env` file under a designated tag using recipients defined in `.env-share`:

```sh
envo push <tag>
```

### Pull Environment Secrets

Fetch and decrypt environment secrets published under a tag, writing the output to `.env`.

When pulling a tag for the first time, specify the publisher's public key using `--owner` to pin the trusted source:

```sh
envo pull <tag> --owner <npub>
```

Subsequent pulls for the same tag use the pinned owner automatically:

```sh
envo pull <tag>
```

## Project Structure

```
.
├── main.rs              # CLI entry point and argument parsing
├── env_files.rs         # `.env` and `.env-share` file parsing
├── event_content.rs     # Payload structure for Nostr event content
├── helper.rs            # Helper module declarations
├── key_gen.rs           # Keypair generation, loading, and prompt management
├── key_valid.rs         # Bech32 key validation and directory path resolution
├── log.rs               # Terminal status logging utilities
├── nostr.rs             # Nostr event building, encryption, fetching, and publishing
├── pull.rs              # Pull command implementation and tag owner resolution
├── push.rs              # Push command implementation and payload encryption
├── relay_provider.rs    # Hardcoded Nostr relay definitions
├── secret_file.rs       # Filesystem permissions enforcement (0600/0700)
├── trusted_owners.rs    # Tag owner pinning logic
├── install.sh           # Unix installation script
└── install.ps1          # Windows PowerShell installer
```