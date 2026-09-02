# envo

A command-line tool for securely sharing environment variables using [Nostr](https://nostr.com)'s [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) encryption. Encrypt secrets for specific recipients, publish to decentralized Nostr relays, and decrypt on authorized machines.

---

## Key Features

- **End-to-End Encryption**: NIP-44 encrypts `.env` files for specified recipients  
- **Decentralized Network**: Publishes and retrieves secrets via public Nostr relays (no central servers)  
- **Trust Management**: Requires explicit trust verification for first-time secret pulls, with global trust mapping  
- **Secure Key Storage**: Stores Nostr keypairs in `~/.envo/keys.json` with strict file permissions (0600/0700)  

---

## Installation

### Unix (Linux/macOS)
```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```
*Optional environment variables:*
- `ENVO_VERSION`: Specify a release version
- `ENVO_INSTALL_DIR`: Set a custom installation path (default: `$HOME/.local/bin`)

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
This creates a Nostr keypair in `~/.envo/keys.json` and displays your public key (`npub...`).

2. **Prepare Project Files**
- `.env`: Secret key-value pairs (e.g., `API_KEY=abc123`)
- `.env-share`: Comma-separated list of recipient public keys (npub/hex format)

Example `.env-share`:
```
npub1abcde...,npub1fghij...
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
envo pull my-project-staging --owner npub1xyz...
```

*Subsequent pulls for the same tag do not require `--owner` after initial verification.*

---

## File Structure

### Project Files
- `.env`: Secrets to share
- `.env-share`: Recipient public keys list

### Global Configuration
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