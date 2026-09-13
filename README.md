# envo

A CLI tool for encrypting and sharing environment files over the Nostr protocol. It lets you publish secrets under a tag and pull them on another machine, using Nostr relays and NIP-44 encryption.

## Table of Contents

- Description
- Features
- Requirements
- Installation
- Usage
- Project Structure
- Tests

## Description

`envo` manages encrypted `.env` distribution via Nostr. You publish secrets under a tag to selected recipients, then pull and decrypt them on a target machine. The tool handles key generation, NIP-44 encryption, relay selection, and trust pinning for tag owners.

## Features

- Generate and store a local Nostr identity (`envo keygen`).
- Push encrypted environment contents to Nostr relays under a tag (`envo push <tag>`).
- Pull and decrypt secrets published under a tag (`envo pull <tag>`).
- Pin and remember the trusted owner (npub) for each tag to prevent unauthorized pulls.
- Restrict key and trust files to owner-only filesystem permissions.

## Requirements

- Rust toolchain (to build from source).
- `curl` or `wget` (for the prebuilt binary installer).
- `unzip` or `tar` (for unpacking release archives).
- A SHA-256 utility (optional; installs proceed without verification if absent).
- Nostr SDK dependency (used internally).

## Installation

### Prebuilt Binary

The installer downloads the matching binary from the latest GitHub release, verifies its checksum, and places it on your PATH.

curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh

Set `ENVO_VERSION` to install a specific release tag. Set `ENVO_INSTALL_DIR` to override the default install directory (`$HOME/.local/bin`).

On Windows, the PowerShell installer is preferred so the binary lands on the Windows PATH:

irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex

### From Source

Build the project with Cargo:

cargo build --release

## Usage

### Key Generation

Create or display your local Nostr identity.

envo keygen

If no valid identity exists, it prompts to generate one. Existing identities are reported and not overwritten.

### Push Secrets

Encrypt and publish `.env` contents under a tag for trusted recipients listed in `.env-share`.

envo push <tag>

The command reads `.env` and `.env-share` from the current working directory. `.env-share` contains comma-separated npubs of trusted recipients. The publisher is automatically included as a recipient.

### Pull Secrets

Fetch and decrypt secrets published under a tag. The first pull for a tag requires `--owner <npub>` to establish trust.

envo pull <tag> [--owner <npub>]

Subsequent pulls for the same tag use the stored owner without requiring `--owner`.

## Project Structure

| File | Purpose |
|------|---------|
| `main.rs` | CLI entry point; parses commands and delegates to subcommands. |
| `commands/` | Subcommand implementations (`key_gen.rs`, `push.rs`, `pull.rs`). |
| `helper.rs` | Module exports. |
| `helper/env_files.rs` | Loads `.env` and `.env-share` from the current directory. |
| `helper/event_content.rs` | Structures Nostr event content (`EventContent`). |
| `helper/key_valid.rs` | Validates keypairs and manages `~/.envo` directory and key file. |
| `helper/log.rs` | Terminal output helpers (step, success, warn, fail). |
| `helper/relay_provider.rs` | Returns the default Nostr relay URLs. |
| `helper/secret_file.rs` | Writes files with owner-only permissions and restricts existing paths. |
| `helper/trusted_owners.rs` | Reads and writes the per-tag owner pin map (`~/.envo/trusted_owners.json`). |
| `nostr.rs` | Nostr SDK modules (`build_and_sign_event`, `encrypter`, `fetch_event`, `publish_event`). |
| `tests.rs` | Test module entry point. |

## Tests

Run the test suite with Cargo:

cargo test

## Limitations

- Prebuilt binaries are only available for Linux (x86_64), macOS (x86_64 and arm64), and Windows (x86_64). Other architectures require building from source.
- The installer skips checksum verification if no SHA-256 utility is present.
- On Windows, a binary installed to `$HOME/.local/bin` may not be on the PATH for PowerShell or cmd; use the PowerShell installer instead.