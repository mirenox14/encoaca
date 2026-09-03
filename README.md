# envo

`envo` is a command-line tool for encrypting and sharing project environment configurations (`.env`) over the Nostr network using NIP-44 encryption.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Default Relays](#default-relays)

## Features

- **Nostr Key Management**: Generate or verify a local Nostr keypair stored securely on disk.
- **Encrypted Sharing**: Encrypt `.env` contents specifically for recipient Nostr public keys listed in `.env-share`.
- **Identity Pinning**: Pin tag publishers by public key (`npub`) to prevent rogue event substitution during secret pulls.
- **File System Permissions**: Enforces strict Unix file permissions (`0600` for key/pin files, `0700` for storage directories).

## Installation

### Shell Installer (Linux / macOS / Git Bash)

Run the installer script to download the binary for your platform and place it in your path:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

You can customize the installation using environment variables:

- `ENVO_VERSION`: The release tag to install (defaults to `latest`).
- `ENVO_INSTALL_DIR`: The destination directory for the binary (defaults to `$HOME/.local/bin`).

### PowerShell Installer (Windows)

On Windows, run the PowerShell installation script:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

## Configuration

`envo` uses local configuration files and project files:

### Local Identity & Storage

- `~/.envo/keys.json`: Stores your Nostr keypair (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Tracks pinned publisher public keys (`npub`) associated with specific pull tags.

### Project Files

When running `envo push`, the binary expects two files in your current working directory:

- `.env`: The secret environment variables to be shared.
- `.env-share`: A comma-separated list of Nostr public keys (`npub`) authorized to decrypt the shared secrets.

## Usage

### Generate or Display Identity

Initialize a local identity or view your existing public key:

```sh
envo keygen
```

If no keys are found, you will be prompted to generate a new pair.

### Push Secrets

Encrypt and publish the local `.env` file to Nostr relays under a specified tag:

```sh
envo push <tag>
```

This command reads `.env` and `.env-share` from the current working directory, encrypts the secrets for every recipient listed in `.env-share` (as well as your own public key), and broadcasts the encrypted payload.

### Pull Secrets

Fetch, decrypt, and save secrets from relays into a local `.env` file:

#### First time pulling a tag
When pulling a tag for the first time, specify the publisher's public key using `--owner`:

```sh
envo pull <tag> --owner <npub>
```

The owner's `npub` will be remembered and pinned to `<tag>` in `~/.envo/trusted_owners.json`.

#### Subsequent pulls
Once a tag's owner is pinned, you can pull updates without specifying `--owner`:

```sh
envo pull <tag>
```

## Default Relays

`envo` publishes and fetches events from the following default Nostr relays:

- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`