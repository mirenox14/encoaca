# envo

`envo` is a command-line tool for securely sharing project environment variables across devices and team members over Nostr relays using NIP-44 encryption.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration Files](#configuration-files)
- [Usage](#usage)
- [Project Structure](#project-structure)

## Features

- **Nostr Identity Management**: Generate and store local Nostr keypairs (`npub`/`nsec`) with file permission protections.
- **Encrypted Secret Sharing**: Read secrets from `.env` and encrypt them individually for recipient public keys listed in `.env-share` using NIP-44 encryption.
- **Relay Publishing**: Publish encrypted secret payloads under tag identifiers to default Nostr relays (`relay.damus.io`, `nos.lol`, `purplepag.es`, `relay.primal.net`, `relay.nostr.band`).
- **Owner Pinning**: Pin trusted publisher public keys per tag to `~/.envo/trusted_owners.json` to prevent untrusted event overwrites.
- **Secure Local Storage**: Enforce restricted Unix permissions (`0700` for directory, `0600` for identity and pin files).

## Installation

### Shell Installer (Linux / macOS)

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

Installation behavior can be customized using environment variables:

- `ENVO_VERSION`: Release tag to install (default: `latest`).
- `ENVO_INSTALL_DIR`: Target directory for the binary (default: `$HOME/.local/bin`).

### PowerShell Installer (Windows)

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### Build from Source

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration Files

| File | Location | Description |
| --- | --- | --- |
| `.env` | Current Working Directory | Contains secret environment variables read by `push` and written by `pull`. |
| `.env-share` | Current Working Directory | Contains a comma-separated list of recipient Nostr public keys. |
| `keys.json` | `~/.envo/keys.json` | Stores your local Nostr keypair (`npub` and `nsec`). |
| `trusted_owners.json` | `~/.envo/trusted_owners.json` | Stores pinned publisher public keys associated with tag names. |

## Usage

### Generate an Identity

Generate a new identity keypair or display the active identity:

```sh
envo keygen
```

### Push Secrets

Encrypt and publish the local `.env` file for a specified tag to all recipients listed in `.env-share` (including your own keypair):

```sh
envo push <TAG>
```

### Pull Secrets

Fetch and decrypt secrets published under a tag:

```sh
envo pull <TAG> --owner <OWNER_NPUB>
```

The `--owner` flag is required the first time a tag is pulled to pin the trusted publisher. Subsequent pulls for the same tag remember the pinned owner:

```sh
envo pull <TAG>
```

## Project Structure

- `main.rs`: Entry point and CLI subcommand dispatcher (`keygen`, `push`, `pull`).
- `key_gen.rs`: Prompts, generates, stores, and validates local Nostr keypairs.
- `key_valid.rs`: Validates key formats and resolves `~/.envo` directory paths.
- `env_files.rs`: Loads and parses local `.env` and `.env-share` files.
- `push.rs`: Encrypts `.env` contents for all target recipients and dispatches Nostr events.
- `pull.rs`: Fetches events by tag from Nostr relays, checks pinned owner trust, decrypts payload, and outputs `.env`.
- `trusted_owners.rs`: Manages tag owner pin entries in `~/.envo/trusted_owners.json`.
- `secret_file.rs`: Utilities for applying strict file modes (`0600`/`0700`) on secret files.
- `relay_provider.rs`: Configures default Nostr relay endpoints.
- `event_content.rs`: Defines data structures for encrypted recipient payload mappings.
- `install.sh`: Shell setup script for Unix systems.
- `install.ps1`: PowerShell setup script for Windows systems.