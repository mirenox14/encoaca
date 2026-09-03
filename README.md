# envo

`envo` is a command-line tool that securely shares encrypted environment variables across teammates over Nostr relays using NIP-44 encryption.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [CLI Reference](#cli-reference)
- [Project Structure](#project-structure)

## Features

- **Nostr Keypair Identity:** Generates and manages local `npub`/`nsec` identities saved to `~/.envo/keys.json`.
- **Targeted NIP-44 Encryption:** Encrypts `.env` secret files specifically for the recipients listed in `.env-share` along with the publisher's key.
- **Relay-Based Distribution:** Publishes encrypted payload events to Nostr relays (`wss://relay.damus.io`, `wss://nos.lol`, `wss://purplepag.es`, `wss://relay.primal.net`, `wss://relay.nostr.band`).
- **Owner Pinning:** Securely pins tag publishers in `~/.envo/trusted_owners.json` to prevent unauthorized tag overwrites on public relays.
- **Strict File Permissions:** Automatically restricts local key files and directories to owner-only permissions (`0600` for files, `0700` for directories on Unix).

## Requirements

- **Linux** (x86_64) or **macOS** (x86_64, arm64 / Apple Silicon).
- `curl` or `wget` (for script installation).
- `unzip` or `tar` depending on OS release target.
- Cargo toolchain (if building from source).

## Installation

### Shell Script

Run the installation script to download prebuilt binaries:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

You can customize the installation using environment variables:

- `ENVO_VERSION`: Release tag to install (defaults to `latest`).
- `ENVO_INSTALL_DIR`: Target installation directory (defaults to `$HOME/.local/bin`).

### From Source

Build and install `envo` directly via Cargo:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

`envo` uses local configuration files in your home directory and project-level files in your working directory.

### Global Configuration (`~/.envo/`)

- `~/.envo/keys.json`: Stores your Nostr identity keypair (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Contains JSON mappings between tag names and trusted publisher `npub` public keys.

### Project Files

When running `envo push`, the following files must exist in the directory where the command is executed:

- `.env`: Contains the secret lines/environment variables to be encrypted and shared.
- `.env-share`: Contains a comma-separated list of recipient Nostr public keys (`npub`) authorized to decrypt the secrets.

## Usage

### 1. Initialize an Identity

Generate or display your Nostr identity:

```sh
envo keygen
```

### 2. Push Secrets to Relays

Ensure `.env` and `.env-share` exist in your project root, then publish the secrets under a tag name:

```sh
envo push my-project-tag
```

### 3. Pull Secrets from Relays

When pulling secrets for a tag for the first time, specify the publisher's public key using `--owner`:

```sh
envo pull my-project-tag --owner npub1...
```

Subsequent pulls remember the trusted owner and write directly to `.env`:

```sh
envo pull my-project-tag
```

## CLI Reference

### `envo keygen`

Generates a new local Nostr keypair if one does not exist, or displays the public key (`npub`) of your existing identity.

### `envo push <TAG>`

Reads `.env` and `.env-share` from the current directory, encrypts `.env` for each recipient public key listed in `.env-share` plus your own, and publishes the signed event under `<TAG>` to default Nostr relays.

### `envo pull <TAG> [--owner <NPUB>]`

Fetches published events under `<TAG>` from relays, verifies the event publisher matches the pinned owner (or sets the pinned owner if `--owner` is provided), decrypts the content, and writes the resulting secrets to `.env`.

## Project Structure

```
.
├── main.rs              # CLI entry point and argument handling
├── env_files.rs         # `.env` and `.env-share` parsing logic
├── event_content.rs     # Data structures for Nostr event content payloads
├── helper.rs            # Helper module declarations
├── install.ps1          # Windows installer script
├── install.sh           # Unix shell installer script
├── key_gen.rs           # Nostr identity generation and verification
├── key_valid.rs         # Directory pathing and key pair validation
├── log.rs               # Console status formatting
├── nostr.rs             # Nostr module exports
├── pull.rs              # Event fetching and decryption logic
├── push.rs              # Event encryption and relay publishing logic
├── relay_provider.rs    # Default Nostr relay URLs list
├── secret_file.rs       # Filesystem restriction and secret file writers
├── tests.rs             # Test module declaration
└── trusted_owners.rs    # Per-tag publisher pinning logic
```