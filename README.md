# envo

A CLI tool for encrypting and sharing environment files via the Nostr protocol. You use it to publish secrets under a tag and pull them back on trusted relays.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Tests](#tests)

## Features

- Generate and manage a local Nostr keypair.
- Encrypt `.env` contents for a list of trusted Nostr public keys and publish under a tag.
- Fetch and decrypt `.env` contents published under a tag by a trusted owner.
- Remember trusted owners per tag to prevent guessing.
- Restrict key and owner files to owner-only filesystem permissions.

## Requirements

- A Unix-like system, or Windows with the PowerShell installer.
- `curl` or `wget` to download prebuilt binaries.
- `sha256sum`, `shasum`, `tar`, `unzip`, or `unzip` to verify and unpack releases.

## Installation

### Prebuilt Binary

Download the matching binary from the latest GitHub release. The install script detects your OS and CPU, verifies the checksum, and places the binary on your PATH.

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

Set the `ENVO_VERSION` environment variable to install a specific release tag. Set `ENVO_INSTALL_DIR` to choose the installation directory (default: `$HOME/.local/bin`).

On Windows, use the PowerShell installer to place the binary on the Windows PATH:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### From Source

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

### Identity

Run `envo keygen` to create or display a Nostr identity. The keypair is stored at `~/.envo/keys.json`. The file is created with owner-only permissions. If valid keys already exist, `envo keygen` displays the public key and exits.

### Project Files

`push` and `pull` read from two files in the directory where you run the command:

- `.env` — the secrets to share.
- `.env-share` — a comma-separated list of trusted Nostr public keys (`npub1...`) allowed to read the secrets.

Both files are required. Missing or unreadable files produce an error.

### Trusted Owners

`pull` requires a trusted owner for each tag. The first time you pull a tag, provide the owner's `npub` with `--owner`:

```sh
envo pull <tag> --owner <npub>
```

The owner is stored in `~/.envo/trusted_owners.json` and reused on subsequent pulls for that tag.

## Usage

### Key Generation

```sh
envo keygen
```

### Push Secrets

Encrypt the contents of `.env` for every key listed in `.env-share` (including yourself) and publish the ciphertext under a tag.

```sh
envo push <tag>
```

### Pull Secrets

Fetch and decrypt `.env` contents published under `<tag>` by its trusted owner.

```sh
envo pull <tag> [--owner <npub>]
```

The decrypted contents are written to `.env` in the current directory.

## Project Structure

| File | Purpose |
|------|---------|
| `main.rs` | Entry point. Parses CLI arguments and dispatches to `keygen`, `push`, or `pull`. |
| `commands/` | Subcommand implementations. |
| `key_gen.rs` | Generates or reads the local Nostr keypair. |
| `key_valid.rs` | Validates keys and restricts filesystem permissions. |
| `push.rs` | Encrypts `.env` contents and publishes them under a tag. |
| `pull.rs` | Fetches events, decrypts content for the local key, and writes `.env`. |
| `env_files.rs` | Loads and parses `.env` and `.env-share`. |
| `event_content.rs` | Defines the `EventContent` struct for Nostr events. |
| `relay_provider.rs` | Returns the default list of Nostr relay URLs. |
| `trusted_owners.rs` | Reads and writes the per-tag owner pin map. |
| `secret_file.rs` | Handles owner-only file creation and permission restriction. |
| `log.rs` | Terminal output helpers (`step`, `success`, `warn`, `fail`). |
| `nostr.rs` | Nostr SDK modules for event building, signing, encryption, fetching, and publishing. |
| `tests.rs` | Test module registration. |

## Tests

Run the test suite with:

```sh
cargo test
```

## Limitations

- Prebuilt binaries are only available for `x86_64` Linux, `x86_64` and `arm64` macOS, and `x86_64` Windows. Other architectures require building from source.
- The security model depends entirely on possession of the local keypair. If the key file is readable by other accounts, the secrets are compromised.