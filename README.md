# envo

`envo` is a command-line tool for sharing and pulling encrypted environment secrets across Nostr relays using NIP-44 encryption.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Generate Identity](#generate-identity)
  - [Push Secrets](#push-secrets)
  - [Pull Secrets](#pull-secrets)
- [CLI Reference](#cli-reference)
- [Project Files and Storage](#project-files-and-storage)
- [Project Structure](#project-structure)

## Features

- **Nostr Relay Integration**: Encrypts and publishes secret payloads over public Nostr relays.
- **NIP-44 Recipient Encryption**: Encrypts environment data separately for each authorized public key (`npub`).
- **Owner Trust Pinning**: Remembers trusted publisher keys per tag in `~/.envo/trusted_owners.json` to prevent payload substitution.
- **File Permission Enforcement**: Automatically restricts access permissions on key and configuration files (`0600` for secret files, `0700` for directories on Unix).

## Installation

### Shell Script (Linux / macOS)

Run the installation script to download and unpack the latest release binary into `$HOME/.local/bin`:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

### PowerShell (Windows)

Install using PowerShell:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### Build from Source

Build and install directly with Cargo:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

### Environment Variables

The shell installer supports the following environment variables:

| Variable | Description | Default |
| --- | --- | --- |
| `ENVO_VERSION` | Release tag to download | `latest` |
| `ENVO_INSTALL_DIR` | Installation target directory | `$HOME/.local/bin` |

### Default Relays

`envo` connects to the following Nostr relays:

- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`

## Usage

### Generate Identity

Generate a local Nostr keypair before performing push or pull operations:

```sh
envo keygen
```

If an identity already exists in `~/.envo/keys.json`, `envo keygen` reports the location and public key without overwriting it.

### Push Secrets

`envo push` requires two files in the execution directory:
- `.env`: Line-separated secret values to share.
- `.env-share`: Comma-separated list of recipient `npub` public keys allowed to read the secrets.

Publish the secrets under a tag:

```sh
envo push <TAG>
```

### Pull Secrets

To fetch and decrypt secrets published under a tag:

1. On the first pull for a tag, pass the `--owner` option to trust and pin the publisher's public key:

```sh
envo pull <TAG> --owner <NPUB>
```

2. Run subsequent pulls for the same tag without specifying `--owner`:

```sh
envo pull <TAG>
```

Upon successful decryption, `envo` writes the secrets to `.env`.

## CLI Reference

### `envo keygen`

Generates a new local Nostr identity (`npub`/`nsec`) or displays the public key of an existing valid keypair stored at `~/.envo/keys.json`.

### `envo push <tag>`

Reads `.env` and `.env-share` from the current directory, encrypts `.env` for each recipient key in `.env-share` and the publisher's key, and publishes the payload to configured relays under `<tag>`.

### `envo pull <tag> [--owner <owner>]`

Fetches events for `<tag>` published by the specified or pinned owner key. Decrypts the recipient payload addressed to the local user key and writes the output to `.env`.

## Project Files and Storage

- `~/.envo/keys.json`: Stores the local Nostr identity (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Stores tag-to-owner `npub` pin mappings.
- `.env`: Source secret file read by `push` and target output file written by `pull`.
- `.env-share`: Local configuration file containing comma-separated recipient public keys.

## Project Structure

```
.
├── env_files.rs          # Loads and parses local .env and .env-share files
├── event_content.rs      # Defines EventContent structure for payload serialization
├── helper.rs             # Helper module exports
├── install.ps1           # Windows PowerShell installer script
├── install.sh            # Unix shell installer script
├── key_gen.rs            # Handles identity generation, storage, and retrieval
├── key_valid.rs          # Keypair validation and .envo directory initialization
├── log.rs                # Terminal transcript and status logging routines
├── main.rs               # CLI argument parsing and entry point
├── nostr.rs              # Nostr event creation, encryption, and relay interactions
├── pull.rs               # Secret fetching, owner verification, and NIP-44 decryption
├── push.rs               # Payload encryption and relay publishing logic
├── relay_provider.rs     # Default Nostr relay endpoint list
├── secret_file.rs        # Filesystem permissions and owner-restricted writes
├── tests.rs              # Test declarations
└── trusted_owners.rs     # Manages tag owner pin storage in ~/.envo/trusted_owners.json
```