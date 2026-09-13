# envo

`envo` is a command-line tool for encrypting and sharing environment variables across teams using Nostr keypairs and relays.

## Features

- **Identity Management**: Generates and manages local Nostr keypairs (`npub`/`nsec`) with Unix file permission safeguards (`0600`/`0700`).
- **NIP-44 Encryption**: Encrypts `.env` files separately for each recipient listed in `.env-share` as well as the publisher.
- **Relay Publishing**: Spreads encrypted payloads across multiple Nostr relays under specified topic tags.
- **Owner Pinning**: Pinpoints and remembers the trusted publisher key for each tag in `~/.envo/trusted_owners.json` to protect against unauthorized updates.

## Installation

### Shell Script (macOS, Linux, Windows)

Install prebuilt binaries via `install.sh`:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

You can configure the installation directory and version using environment variables:

- `ENVO_VERSION`: The release tag to install (defaults to `latest`).
- `ENVO_INSTALL_DIR`: Target directory for the executable (defaults to `$HOME/.local/bin`).

### PowerShell (Windows)

On Windows systems, install using PowerShell:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### Build from Source

Install directly from the Git repository using Cargo:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

## Configuration

`envo` uses local configuration files stored in the user home directory as well as files inside your project directory.

### Global Configuration (`~/.envo/`)

- `~/.envo/keys.json`: Stores the user's Nostr public and private keys (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Maps topic tags to trusted publisher public keys (`npub`).

### Project Files

Running `envo push` requires two files present in the current working directory:

- `.env`: Contains the key-value environment pairs to be shared.
- `.env-share`: Contains a comma-separated list of recipient Nostr public keys (`npub` or hex format).

## Usage

### Generate or Display Identity

Create a new local identity or show the existing public key:

```sh
envo keygen
```

Identity keys are written to `~/.envo/keys.json`.

### Push Secrets

Encrypt `.env` for all recipients listed in `.env-share` and publish the payload under a tag:

```sh
envo push <TAG>
```

Secrets are published to the following relays:
- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`

### Pull Secrets

Fetch and decrypt published secrets for a tag, writing the output directly to `.env`:

```sh
envo pull <TAG> --owner <NPUB>
```

The `--owner` flag specifies the publisher's `npub`. Once set, the publisher key is pinned in `~/.envo/trusted_owners.json`, allowing future pulls to omit the flag:

```sh
envo pull <TAG>
```