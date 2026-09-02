# envo

A command-line tool for securely sharing environment variables using Nostr's decentralized network and NIP-44 encryption. Encrypt secrets for specific recipients, publish to Nostr relays, and decrypt them on authorized machines.

---

## Key Features

- **NIP-44 Encryption**: End-to-end encryption for `.env` files using Nostr's standard encryption protocol
- **Decentralized Distribution**: Publish and fetch encrypted secrets via public Nostr relays without central servers
- **Owner Verification**: Trust-based system using `~/.envo/trusted_owners.json` to prevent unauthorized updates
- **Secure Key Management**: Stores Nostr keypairs in `~/.envo/keys.json` with strict file permissions (0600/0700)

---

## Installation

### Unix (Linux/macOS)
```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

*Optional:*
- `ENVO_VERSION`: Specify release version
- `ENVO_INSTALL_DIR`: Set custom installation path (default: `$HOME/.local/bin`)

### Windows
```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### From Source
```sh
cargo install --git https://github.com/kaihere14/climenv
```

---

## Getting Started

1. **Generate Identity**
```sh
envo keygen
```
Creates a Nostr keypair in `~/.envo/keys.json` and displays your public key (`npub...`).

2. **Prepare Files**
- `.env`: Secret key-value pairs
- `.env-share`: Comma-separated list of recipient public keys (npub/hex)

Example `.env-share`:
```
npub1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx,
npub1yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

---

## Usage

### Push Secrets
```sh
envo push <tag>
```
Encrypts `.env` for all keys in `.env-share` and broadcasts to Nostr relays.

Example:
```sh
envo push my-project-staging
```

### Pull Secrets
```sh
envo pull <tag> --owner <owner-npub>
```
Decrypts secrets published under the specified tag. First-time use requires `--owner` to establish trust.

Example:
```sh
envo pull my-project-staging --owner npub1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Subsequent pulls for the same tag don't need `--owner` after initial verification.

---

## File Structure

- **Project Files**
  - `.env`: Secrets to share
  - `.env-share`: Authorized recipients list

- **Global Configuration**
  - `~/.envo/keys.json`: Nostr keypair storage
  - `~/.envo/trusted_owners.json`: Tag-to-owner trust mapping

---

## Default Relays

Events are published to and fetched from these relays by default:
- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`