# envo

`envo` encrypts local environment variables and publishes them to Nostr relays under named tags. It is designed for individuals and small teams who need to share secrets without a central server.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [CLI Reference](#cli-reference)
- [Project Structure](#project-structure)
- [Tests](#tests)

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

Download prebuilt binaries from the [kaihere14/climenv releases page](https://github.com/kaihere14/climenv) or install from source.

### Linux

```bash
curl -fsSL https://github.com/kaihere14/climenv/releases/latest/download/envo-x86_64-unknown-linux-gnu.tar.gz | tar -xz -C ~/.local/bin
```

### macOS

```bash
curl -fsSL https://github.com/kaihere14/climenv/releases/latest/download/envo-x86_64-apple-darwin.tar.gz | tar -xz -C ~/.local/bin
```

### Windows

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### From Source

```bash
cargo install --git https://github.com/kaihere14/climenv
```

## Usage

### Generate an Identity

Run `keygen` to create a local identity:

```bash
envo keygen
```

This creates `~/.envo/keys.json` if it does not already exist. Re-running the command displays the existing identity.

### Publish Secrets

1. Create a `.env` file containing the secrets to share.
2. Create a `.env-share` file containing recipient Nostr `npub` keys, one per line.
3. Run:

```bash
envo push <tag>
```

`envo` reads `.env` and `.env-share` from the current working directory, encrypts the secrets for each recipient listed in `.env-share`, and publishes the payload as a Nostr event under the specified tag.

### Retrieve Secrets

To fetch and decrypt secrets for a tag:

```bash
envo pull <tag> --owner <npub>
```

`envo` fetches and decrypts the matching event using your local Nostr identity and writes the output to `.env`.

The `--owner` flag is required on the first pull for a tag. The tag owner is saved in `~/.envo/trusted_owners.json` so that subsequent pulls for the same tag use the stored owner automatically. If no owner is stored and `--owner` is omitted, the command fails and prompts for the owner flag.

## CLI Reference

```text
envo [COMMAND]

Commands:
  keygen     Generate or report a local Nostr identity.
  push <tag> Publish encrypted secrets under a tag.
  pull <tag> Fetch and decrypt secrets for a tag.
```

## Project Structure

| File | Purpose |
| --- | --- |
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

Run the test suite using Cargo:

```bash
cargo test
```