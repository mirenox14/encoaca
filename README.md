# envo

`envo` is a command-line tool for encrypting and sharing environment variables across teams using the Nostr network. It encrypts local `.env` contents for specified recipient public keys and publishes or retrieves them from Nostr relays.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Default Relays](#default-relays)

## Features

- Generates and manages local Nostr identities (`npub`/`nsec`) stored in restricted local files.
- Encrypts environment variable files using NIP-44 ECDH for multiple team members.
- Publishes encrypted secret payloads to Nostr relays under custom tags.
- Retrieves and decrypts environment files from pinned trusted tag owners.

## Installation

### Shell (Linux / macOS)

Run the shell installer script:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

You can customize the installation using environment variables:

```sh
export ENVO_VERSION="latest"
export ENVO_INSTALL_DIR="$HOME/.local/bin"
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

### PowerShell (Windows)

Run the PowerShell installer script:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### From Source

Build and install directly using Cargo:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

`envo` operates using project files in your current working directory and local configuration files in your home directory.

### Project Files

- `.env`: Contains line-separated key-value environment variables to encrypt and publish.
- `.env-share`: Contains a comma-separated list of trusted recipient Nostr public keys (`npub...`) allowed to decrypt the pushed `.env` contents.

### Local Configuration (`~/.envo/`)

- `keys.json`: Holds your local identity (`npub` and `nsec`). Permissions are restricted to `0700` for the directory and `0600` for the key file on Unix systems.
- `trusted_owners.json`: Stores local pins mapping project tag names to trusted publisher public keys.

## Usage

### Key Generation

Generate a new identity or display your existing public key:

```sh
envo keygen
```

### Pushing Secrets

Publish secrets from the local `.env` file for all public keys listed in `.env-share`:

```sh
envo push <tag>
```

`envo push` reads `.env` and `.env-share`, encrypts the `.env` contents individually for every listed recipient (plus your own key), and publishes the event under `<tag>` to configured relays.

### Pulling Secrets

Fetch and decrypt secrets for a tag published by a specific owner:

```sh
envo pull <tag> --owner <npub>
```

On the first pull for a tag, `--owner` pins the specified publisher key in `~/.envo/trusted_owners.json`.

Subsequent pulls for the same tag omit the `--owner` argument:

```sh
envo pull <tag>
```

When pulled, the decrypted secrets overwrite or create `.env` in your current working directory.

## Default Relays

`envo` connects to the following default Nostr relays:

- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`