# envo

`envo` is a command-line utility for encrypting, publishing, and pulling environment secrets across Nostr relays using NIP-44 encryption. It allows project teams to securely distribute `.env` contents using Nostr keypairs.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Default Relays](#default-relays)
- [Data Storage](#data-storage)

## Features

- **Nostr Key Management**: Generates and manages local Nostr identity keypairs (`npub`/`nsec`).
- **NIP-44 Encryption**: Encrypts secret key-value pairs individually for each recipient public key specified in `.env-share`, as well as for the publisher.
- **Relay Publishing & Retrieval**: Publishes encrypted payload events under custom tags to Nostr relays and pulls tagged secrets back into `.env`.
- **Owner Pinning**: Remembers the trusted publisher (`npub`) for each tag upon initial pull, ensuring subsequent pulls only accept events from that publisher.
- **File Security**: Enforces restricted permissions (`0600` for files, `0700` for directories on Unix systems) on local identity keys and trust configurations.

## Installation

### Shell Script (macOS and Linux)

Run the shell installer to download and install the binary:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

The installer supports the following environment variables:

- `ENVO_VERSION`: Target release tag to install (defaults to `latest`).
- `ENVO_INSTALL_DIR`: Target installation directory (defaults to `$HOME/.local/bin`).

### PowerShell (Windows)

Run the PowerShell installer:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

## Configuration

`envo push` requires two files in the working directory where the command is executed:

1. `.env`: The environment file containing secrets to be shared.
2. `.env-share`: A text file containing a comma-separated list of recipient Nostr public keys (`npub...`) allowed to decrypt the secrets.

Example `.env-share`:

```text
npub1...key1, npub1...key2
```

## Usage

### Generate or Inspect Identity

Generate a new Nostr keypair or inspect an existing identity stored in `~/.envo/keys.json`:

```sh
envo keygen
```

### Push Secrets

Encrypt and publish the `.env` secrets for a target tag to configured relays:

```sh
envo push my-project-tag
```

This reads `.env` and `.env-share`, encrypts the payload for each public key listed (and your own public key), and publishes the tagged event.

### Pull Secrets

Fetch, decrypt, and save secrets to `.env` for a specific tag.

The first time you pull a tag, specify the publisher's public key using `--owner`:

```sh
envo pull my-project-tag --owner npub1...
```

For subsequent pulls of the same tag, the stored owner pin is used automatically:

```sh
envo pull my-project-tag
```

## Default Relays

`envo` connects to the following default Nostr relays:

- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`

## Data Storage

`envo` maintains state inside `~/.envo/`:

- `~/.envo/keys.json`: Stores your generated Nostr keypair (`{"npub": "...", "nsec": "..."}`).
- `~/.envo/trusted_owners.json`: Stores the map of tag names to trusted publisher public keys (`{"<tag>": "<npub>"}`).