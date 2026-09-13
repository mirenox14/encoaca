# envo

`envo` is a command-line tool for encrypting, sharing, and retrieving environment variables over Nostr relays using keypair identities.

## Features

- Generates and manages local Nostr identity keypairs (`npub`/`nsec`).
- Encrypts local `.env` files for recipient public keys listed in `.env-share`.
- Publishes encrypted payloads under a tag to public Nostr relays.
- Fetches and decrypts tagged environment secrets from trusted publishers directly into `.env`.
- Remembers and pins trusted owner public keys per tag in `~/.envo/trusted_owners.json`.
- Automatically restricts file permissions (`0600` for files, `0700` for directories) on identity keys and settings on Unix systems.

## Installation

### Shell Installer (Linux / macOS)

Run the install script to download the prebuilt binary to `$HOME/.local/bin`:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

You can customize the installation using environment variables:

```sh
export ENVO_VERSION="v0.1.0"
export ENVO_INSTALL_DIR="$HOME/bin"
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

### PowerShell Installer (Windows)

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### Build from Source

Install directly using Cargo:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

`envo` operates on project files located in the current working directory, as well as configuration files stored in the user's home directory.

### Project Files

- `.env`: Contains line-separated environment secrets to share or write to.
- `.env-share`: Contains a list of recipient Nostr public keys (`npub`), separated by commas or newlines.

### Global Files

- `~/.envo/keys.json`: Stores your local Nostr identity keypair (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Remembers the trusted publisher (`npub`) pinned for each tag.

## Usage

### 1. Initialize an Identity

Generate a new Nostr identity keypair or view your existing public key:

```sh
envo keygen
```

### 2. Share Secrets (`push`)

Ensure `.env` contains your secrets and `.env-share` lists the recipient public keys. Then publish the encrypted payload under a tag name:

```sh
envo push my-project-tag
```

This encrypts the `.env` content for every valid `npub` in `.env-share` as well as your own public key, then sends the event to default Nostr relays (`wss://relay.damus.io`, `wss://nos.lol`, `wss://purplepag.es`, `wss://relay.primal.net`, `wss://relay.nostr.band`).

### 3. Retrieve Secrets (`pull`)

To pull secrets for the first time, specify the publisher's public key using `--owner`:

```sh
envo pull my-project-tag --owner npub1...
```

Subsequent pulls for the same tag automatically use the pinned owner from `~/.envo/trusted_owners.json`:

```sh
envo pull my-project-tag
```

Upon successful decryption, `envo` writes the secrets directly to `.env` in your current directory.

## CLI Reference

### `envo keygen`

Generates a new local Nostr identity in `~/.envo/keys.json` if one does not exist, or outputs the existing `npub` public key.

### `envo push <tag>`

Reads `.env` and `.env-share` from the current working directory, encrypts the payload for all recipient public keys, and publishes the event to Nostr relays under `<tag>`.

### `envo pull <tag> [--owner <npub>]`

Fetches events tagged with `<tag>` published by the specified owner public key. Decrypts the secret matching the local identity and writes it to `.env`.

- `--owner <npub>`: Specifies the Nostr public key (`npub`) of the publisher to trust for `<tag>`. Required on the initial pull for a tag.