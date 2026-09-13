# envo

A Nostr-based tool for sharing encrypted environment files under named tags. It lets you push secrets to a set of trusted recipients and pull them back on another machine.

## Table of Contents

- Description
- Features
- Requirements
- Installation
- Usage
- Project Structure
- Tests

## Description

`envo` (short for "environment") manages encrypted `.env` secrets using the Nostr protocol. You publish secrets under a tag, and authorized teammates pull them using their Nostr identity. The project stores your keypair at `~/.envo/keys.json`, remembers which pubkey owns each tag in `~/.envo/trusted_owners.json`, and encrypts secrets with NIP-44.

## Features

- Generate and manage a local Nostr identity.
- Push encrypted environment contents to Nostr relays under a tag.
- Pull encrypted secrets from a trusted owner by tag.
- Restrict key and trust files to owner-only filesystem permissions.
- Use a fixed set of relay URLs.

## Requirements

- A POSIX-like environment (the key and trust files are restricted with `0o600` / `0o700` permissions; on non-Unix platforms this is a no-op).
- A working installation of Rust and Cargo to build from source.

## Installation

### From Source

Clone the repository and build with Cargo:

```text
git clone https://github.com/kaihere14/climenv.git
cd climenv
cargo build --release
```

The resulting binary is `target/release/envo`.

### From Prebuilt Binary

Download the installer script and run it:

```text
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

The script detects your OS and architecture, downloads the matching release asset, verifies its SHA-256 checksum, and installs `envo` to `$HOME/.local/bin` by default. Set `ENVO_VERSION` to install a specific release tag, or `ENVO_INSTALL_DIR` to change the destination.

On Windows, use the PowerShell installer instead so the binary is on the Windows PATH:

```text
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

## Usage

Initialize your identity:

```text
envo keygen
```

This creates `~/.envo/keys.json` if no valid keypair exists, or reports the existing public key. The command is safe to re-run.

Push secrets under a tag. The command reads `.env` for the contents and `.env-share` for the list of trusted recipient pubkeys:

```text
envo push <tag>
```

Pull secrets for a tag. The first time you pull a tag, provide the owner's npub with `--owner`:

```text
envo pull <tag> --owner <npub>
```

After the first pull, the owner is remembered and reused.

## Project Structure

| File | Purpose |
|------|---------|
| `main.rs` | Entry point; parses CLI arguments and dispatches to subcommands. |
| `commands/` | Subcommand implementations (defined as modules in `main.rs`). |
| `key_gen.rs` | Identity generation, validation, and loading. |
| `key_valid.rs` | Keypair validation and filesystem permission helpers. |
| `pull.rs` | Fetches and decrypts secrets for a tag from Nostr relays. |
| `push.rs` | Encrypts `.env` contents and publishes them under a tag. |
| `env_files.rs` | Reads `.env` and `.env-share` from the current working directory. |
| `event_content.rs` | `EventContent` struct for Nostr event payloads. |
| `relay_provider.rs` | Hardcoded list of relay URLs. |
| `secret_file.rs` | Owner-only file and directory permission logic. |
| `trusted_owners.rs` | Per-tag owner pinning in `~/.envo/trusted_owners.json`. |
| `log.rs` | Terminal output helpers (step, success, warn, fail). |
| `nostr.rs` | Nostr SDK modules for event building, signing, encryption, and publishing. |
| `tests.rs` | Test entry point. |

## Tests

Run the test suite with Cargo:

```text
cargo test
```