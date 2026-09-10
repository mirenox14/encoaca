
# envo

A command-line tool for publishing and retrieving encrypted environment files via the Nostr protocol. It stores secrets in `.env` and trusts publishers through a per-tag pinned public key.

## Table of Contents

- Description
- Features
- Requirements
- Installation
- Usage
- Project Structure
- Tests
## Description

`envo` manages secrets through Nostr tags. Publish a tagged set of environment variables with `envo push <tag>`, and retrieve them on another machine with `envo pull <tag>`.

Trust is established per tag by pinning a publisher's public key with `--owner <npub>` during the first pull.

## Features

- Generate and manage a local Nostr identity with `envo keygen`.
- Encrypt `.env` contents and publish them under a tag with `envo push <tag>`.
- Fetch and decrypt tagged secrets into `.env` with `envo pull <tag>`.
- Restrict key files and the configuration directory to owner-only permissions on Unix.
- Verify downloaded binaries with SHA-256 checksums.

## Requirements

- A Unix-like operating system or Windows with a compatible Rust toolchain.
- The `nostr-sdk` crate.

## Installation

Download a prebuilt binary from the latest GitHub release.

```sh
curl -fsSL https://github.com/mirenox14/encoaca/releases/latest/download/envo-x86_64-unknown-linux-gnu.tar.gz | tar -xz
```

Verify the checksum before running the binary:

```sh
sha256sum envo
```

Alternatively, build from source:

```sh
cargo install --git https://github.com/mirenox14/encoaca
```
## Usage

### Generate an identity

Create or display your Nostr keypair.

```sh
envo keygen
```

If no valid keys exist, the command prompts to generate a new identity and stores it in `~/.envo/keys.json`.

### Publish secrets

Read `.env` and `.env-share` from the current directory, encrypt the contents for each trusted pubkey listed in `.env-share`, and publish under a tag.

```sh
envo push <tag>
```

### Retrieve secrets

Fetch and decrypt secrets published under `<tag>` by its trusted owner into `.env`. Provide the owner's `npub` on the first pull for a new tag.

```sh
envo pull <tag> --owner <npub>
```

Subsequent pulls for the same tag use the stored pin.

### CLI Reference

```
envo [command]

Commands:
  keygen             Generate or display the local Nostr identity
  push <tag>         Publish encrypted environment files under a tag
  pull <tag> [--owner <npub>]
                     Fetch and decrypt tagged secrets into .env
```

## Project Structure

| File | Purpose |
|------|---------|
| `main.rs` | CLI entry point using `clap`; dispatches to subcommands |
| `commands/` | Subcommand implementations (implied by `main.rs`) |
| `helper.rs` | Module declarations |
| `key_gen.rs` | Identity generation and validation |
| `key_valid.rs` | Keypair validation and directory/file permission setup |
| `secret_file.rs` | Owner-only file and directory permission enforcement |
| `event_content.rs` | `EventContent` struct for Nostr event payloads |
| `log.rs` | Terminal output helpers (step, success, warn, fail) |
| `nostr.rs` | Nostr SDK integration modules |
| `pull.rs` | `pull` command: fetches and decrypts tagged events |
| `push.rs` | `push` command: encrypts and publishes tagged events |
| `relay_provider.rs` | Default Nostr relay URLs |
| `trusted_owners.rs` | Per-tag owner pin storage and retrieval |
| `tests.rs` | Test module declarations |

## Tests

Run the test suite with:

```sh
cargo test
```

The project includes a `nip44_test` module under `tests.rs`.
