# encoaca

`envo` encrypts local environment variables and publishes them to Nostr relays under named tags. It is for individuals and small teams who need to share secrets without a central server.

## Table of Contents

- Description
- Features
- Requirements
- Installation
- Usage
- Project Structure
- Tests

## Description

`envo` reads `.env` and `.env-share` from the current directory, encrypts the contents for each recipient listed in `.env-share`, and publishes the encrypted payload as a Nostr event under a tag you specify. To retrieve secrets, you run `envo pull` with a tag, which fetches and decrypts the matching event using your Nostr identity.

## Features

- Generate and manage a local Nostr identity with `envo keygen`.
- Encrypt and publish environment secrets to Nostr relays under a tag with `envo push <tag>`.
- Fetch and decrypt secrets for a tag from Nostr relays with `envo pull <tag> [--owner <npub>]`.
- Restrict key files and the configuration directory to owner-only access.
- Pin a trusted Nostr pubkey per tag to prevent unauthorized secret retrieval.

## Requirements

- A working installation of Rust and Cargo.
- Network access to at least one Nostr relay.

## Installation

Download the prebuilt binary for your platform from the [kaihere14/climenv](https://github.com/kaihere14/climenv) GitHub releases.

### Linux

```
curl -fsSL https://github.com/kaihere14/climenv/releases/latest/download/envo-x86_64-unknown-linux-gnu.tar.gz | tar -xz -C ~/.local/bin
```

### macOS

```
curl -fsSL https://github.com/kaihere14/climenv/releases/latest/download/envo-x86_64-apple-darwin.tar.gz | tar -xz -C ~/.local/bin
```

### Windows

```
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

Alternatively, install from source:

```
cargo install --git https://github.com/kaihere14/climenv
```

## Usage

### Generate an identity

```
envo keygen
```

This creates `~/.envo/keys.json` if one does not exist. Re-running the command reports the existing identity.

### Publish secrets

1. Create `.env` with the secrets to share.
2. Create `.env-share` with one Nostr `npub` per line.
3. Run:

```
envo push <tag>
```

The first time you push a tag, you must specify the owner of the tag with `--owner <npub>` when pulling. The owner is remembered in `~/.envo/trusted_owners.json`.

### Retrieve secrets

```
envo pull <tag> --owner <npub>
```

The `--owner` flag is required the first time you pull a tag. Subsequent pulls for the same tag use the stored owner. If no owner is stored, the command fails and asks for `--owner`.

### CLI reference

```
envo [COMMAND]

Commands:
  keygen     Generate or report a local Nostr identity.
  push <tag> Publish encrypted secrets under a tag.
  pull <tag> Fetch and decrypt secrets for a tag.
```

## Project Structure

| File | Purpose |
|------|---------|
| `main.rs` | CLI parsing, command dispatch, and top-level error handling. |
| `key_gen.rs` | Identity generation, storage, and retrieval in `~/.envo/keys.json`. |
| `key_valid.rs` | Validation of Nostr keypairs and permission restriction for the envo directory and key file. |
| `push.rs` | Encrypts `.env` contents for recipients and publishes a Nostr event. |
| `pull.rs` | Fetches events for a tag, decrypts them for the caller, and writes `.env`. |
| `event_content.rs` | Serde-serializable structure for encrypted event content. |
| `relay_provider.rs` | Hardcoded list of Nostr relay WebSocket URLs. |
| `trusted_owners.rs` | Per-tag owner pinning in `~/.envo/trusted_owners.json`. |
| `secret_file.rs` | Filesystem permission helpers for secrets. |
| `log.rs` | Terminal output helpers (step, success, warn, fail). |
| `env_files.rs` | Reads `.env` and `.env-share` from the current working directory. |
| `nostr.rs` | Submodules for event building, signing, encryption, fetching, and publishing. |
| `helper.rs` | Module re-exports. |
| `tests.rs` | Test entry point. |
| `install.sh` | Shell script for downloading and verifying prebuilt binaries. |
| `install.ps1` | PowerShell installer for Windows. |

## Tests

Run the test suite with:

```
cargo test
```