# envo

A command-line tool for secure, decentralized environment variable sharing using [Nostr](https://nostr.com) and [NIP-44](https://github.com/nostr-protocol/nips/blob/master/44.md) encryption. Encrypt secrets for specific recipients, publish to relays, and decrypt on authorized machines with trust-based access control.

---

## Key Features

- **NIP-44 Encryption**: End-to-end encryption for `.env` files with recipient-specific keys  
- **Decentralized Storage**: Publish and retrieve secrets via public Nostr relays  
- **Trust-Based Access**: First-time decryption requires explicit owner verification  
- **Secure File Permissions**: Key files use 0600/0700 permissions by default  

---

## Installation

### Unix (Linux/macOS)
```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```
*Optional parameters:*
- `ENVO_VERSION`: Specify a release version
- `ENVO_INSTALL_DIR`: Set custom path (default: `$HOME/.local/bin`)

### Windows
```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### From Source
```sh
cargo install --git https://github.com/kaihere14/climenv
```

---

## Usage

### 1. Generate Identity
```sh
envo keygen
```
Creates a Nostr keypair in `~/.envo/keys.json` (0600 permissions) and displays your public key (`npub...`).

### 2. Prepare Project Files
- `.env`: Secret key-value pairs (e.g., `API_KEY=abc123`)
- `.env-share`: Comma-separated recipient keys (npub/hex format)

Example `.env-share`:
```text
npub1abcde...,npub1fghij...
```

### 3. Push Secrets
```sh
envo push <tag>
```
Encrypts `.env` for all keys in `.env-share` and broadcasts to Nostr relays.

Example:
```sh
envo push my-project-staging
```

### 4. Pull Secrets
```sh
envo pull <tag> --owner <owner-npub>
```
Decrypts secrets under the specified tag. First-time use requires `--owner` to establish trust in `~/.envo/trusted_owners.json`.

Example:
```sh
envo pull my-project-staging --owner npub1xyz...
```

Subsequent pulls for the same tag use stored trust mappings automatically.

---

## Configuration

### File Structure
- **Project Files**  
  - `.env`: Secrets to share  
  - `.env-share`: Recipient public keys  

- **Global Configuration**  
  - `~/.envo/keys.json`: Nostr keypair (0600)  
  - `~/.envo/trusted_owners.json`: Tag-to-owner trust mappings  

### Default Relays
Events are published to and fetched from:
- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`

---

## Security

- **Key Storage**: All key files use strict 0600/0700 permissions  
- **Trust Verification**: Prevents unauthorized decryption via explicit owner mapping  
- **Network Security**: Secrets are encrypted using NIP-44 before transmission