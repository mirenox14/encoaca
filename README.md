# envo

A command-line tool for publishing and retrieving encrypted project secrets over the Nostr protocol using tagged events. It is for developers who need to share sensitive environment variables with trusted teammates.

## Table of Contents

- Description
- Features
- Requirements
- Installation
- Usage
- Project Structure
- Tests

## Description

`envo` stores secrets in `.env` and distributes them via Nostr relays under user-defined tags. Recipients listed in `.env-share` are encrypted for using NIP-44. You identify yourself with a local Nostr keypair and pin a trusted owner per tag before pulling secrets.

## Features

- Generate and manage a local Nostr identity.
- Publish encrypted secrets under a tag for trusted recipients.
- Fetch and decrypt secrets published under a tag by its trusted owner.
- Restrict file and directory permissions to owner-only on Unix.
- Remember trusted owners per tag in `~/.envo/trusted_owners.json`.

## Requirements

- A Unix-like operating system or Windows with a compatible shell.
- The `nostr_sdk` crate (used internally).
- `curl` or `wget` to download the binary during installation.
- `sha256sum` or `shasum` to verify downloaded binaries.

## Installation

Download the prebuilt binary for your platform from the [kaihere14/climenv](https://github.com/kaihere14/climenv) releases page.

Install with `install.sh` or `install.ps1`:

```bash
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

Or on Windows:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

The installer detects your OS and architecture, downloads the matching release asset, verifies its checksum, and places the `envo` binary on your PATH. If `ENVO_VERSION` is set, it installs that release tag; otherwise, it uses the latest. Set `ENVO_INSTALL_DIR` to change the destination directory.

After installation, verify that the install directory is on your PATH. If it is not, add it to your shell profile:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## Usage

Create an identity before using any command:

```bash
envo keygen
```

This generates a Nostr keypair in `~/.envo/keys.json` and prints your public key. Re-running the command reports the existing identity.

Publish secrets under a tag:

```bash
envo push <tag>
```

This reads `.env` and `.env-share` from the current directory, encrypts the contents for each recipient in `.env-share` (including yourself), and publishes the event to configured relays.

Retrieve secrets published under a tag:

```bash
envo pull <tag>
```

The first time you pull a tag, provide the trusted owner's npub:

```bash
envo pull <tag> --owner <npub>
```

This fetches the event from relays, decrypts the entry addressed to you, and writes it to `.env`. The owner is remembered for subsequent pulls.

## Project Structure

```
├── env_files.rs      # Reads .env and .env-share from the current directory.
├── event_content.rs  # Defines the JSON structure of encrypted event content.
├── helper.rs         # Module declarations.
├── key_gen.rs        # Generates or loads the local Nostr keypair.
├── key_valid.rs      # Validates key format and manages ~/.envo directory/file permissions.
├── log.rs            # Prints status symbols to stdout and stderr.
├── main.rs           # CLI entry point using clap.
├── nostr.rs          # Submodules for event building, signing, encryption, fetching, and publishing.
├── pull.rs           # Fetches and decrypts secrets for a tag.
├── push.rs           # Encrypts and publishes secrets under a tag.
├── relay_provider.rs # Returns the list of relay URLs.
├── secret_file.rs    # Writes files with owner-only permissions.
├── tests.rs          # Test module declarations.
├── trusted_owners.rs # Stores and retrieves the per-tag trusted owner pin.
├── install.sh        # Unix installer script.
├── install.ps1       # Windows installer script.
└── README.md         # This file.
```

## Tests

Run the test suite with:

```bash
cargo test
```

## Limitations

- Prebuilt binaries are available for Linux (x86_64), macOS (x86_64 and arm64), and Windows (x86_64). Other architectures require building from source with `cargo install --git https://github.com/kaihere14/climenv`.
- The default relays are `wss://relay.damus.io`, `wss://nos.lol`, `wss://purplepag.es`, `wss://relay.primal.net`, and `wss://relay.nostr.band`. These are hardcoded in `relay_provider.rs` and are not configurable from the CLI.