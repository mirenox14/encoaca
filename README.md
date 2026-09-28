# envo

A Nostr-based tool for sharing encrypted environment files (`.env`) across teams using tags and trusted pubkeys. It generates and manages Nostr keypairs, pushes secrets to relays, and pulls them back using NIP-44 encryption.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Tests](#tests)

## Features

- Generate and manage a local Nostr identity (`envo keygen`).
- Push encrypted `.env` contents to Nostr relays under a tag (`envo push <tag>`).
- Pull encrypted `.env` contents from a trusted owner by tag (`envo pull <tag>`).
- Encrypt secrets per recipient using NIP-44 and publish to configured relays.
- Restrict filesystem permissions on key and trust files to owner-only.

## Requirements

- A POSIX-compatible system or Windows with a POSIX-compatible shell for the installer.
- `curl` or `wget` to download release binaries.
- `sha256sum` or `shasum` to verify downloads.
- `unzip` on Windows to extract release archives.

## Installation

Download the prebuilt binary from the latest GitHub release using the provided installer script.

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

The script detects your OS and CPU architecture, downloads the matching `envo` binary, verifies its checksum, and places it in `$HOME/.local/bin`. If the directory is not on your PATH, add it to your shell profile:

```sh
export PATH="$HOME/.local/bin:$PATH"
```

On Windows, use the PowerShell installer instead to place the binary on the Windows PATH:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

After installation, create your identity:

```sh
envo keygen
```

## Usage

The CLI accepts three subcommands.

### Generate an identity

Create or display a local Nostr keypair stored in `~/.envo/keys.json`.

```sh
envo keygen
```

### Push secrets

Publish the contents of `.env` to Nostr relays under a tag, encrypted for every pubkey listed in `.env-share` and for yourself.

```sh
envo push <tag>
```

Both `.env` and `.env-share` must exist in the current working directory. `.env` contains the secrets to share. `.env-share` lists the trusted pubkeys, one per line or comma-separated.

### Pull secrets

Fetch and decrypt secrets published under a tag by its trusted owner into `.env`.

```sh
envo pull <tag>
```

On the first pull for a tag, specify the owner's pubkey with `--owner`:

```sh
envo pull <tag> --owner <npub>
```

The owner pubkey is stored in `~/.envo/trusted_owners.json` and remembered for future pulls.

## Project Structure

| File | Purpose |
|------|---------|
| `main.rs` | CLI entry point using `clap`; dispatches to `keygen`, `push`, and `pull`. |
| `key_gen.rs` | Generates or reads the Nostr keypair from `~/.envo/keys.json`. |
| `key_valid.rs` | Validates key format and restricts permissions on the keys directory and file. |
| `push.rs` | Encrypts `.env` contents and publishes an event to relays. |
| `pull.rs` | Fetches events, decrypts secrets addressed to the local pubkey, and writes `.env`. |
| `env_files.rs` | Reads and parses `.env` and `.env-share` from the current directory. |
| `event_content.rs` | Defines the `EventContent` structure for NIP-44 recipients. |
| `relay_provider.rs` | Returns the list of default relay URLs. |
| `trusted_owners.rs` | Stores and retrieves the per-tag trusted owner pin. |
| `secret_file.rs` | Handles owner-only file creation and permission restriction. |
| `log.rs` | Terminal output helpers (`step`, `success`, `warn`, `fail`). |
| `nostr.rs` | Module declarations for event building, signing, encryption, fetching, and publishing. |
| `helper.rs` | Module re-exports. |
| `tests.rs` | Test module declarations. |
| `install.sh` | POSIX installer script. |
| `install.ps1` | Windows installer script. |

## Tests

Run the test suite with Cargo:

```sh
cargo test
```

The project contains a NIP-44 test module under `tests.rs`.