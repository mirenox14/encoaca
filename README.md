# envo

`envo` is a command-line tool for securely sharing environment variables across teams and devices using decentralised Nostr relays and NIP-44 end-to-end encryption.

With `envo`, you can encrypt your local `.env` file for a set of trusted Nostr public keys, publish it under a tag, and pull it on any authorized machine.

---

## Features

- **End-to-End Encryption**: Encrypts secret environment files using Nostr NIP-44 encryption so only intended recipients can read them.
- **Decentralized Relay Support**: Publishes and fetches encrypted payloads through public Nostr relays without requiring a centralized server.
- **Owner Pinning**: Remembers trusted publisher public keys per tag in `~/.envo/trusted_owners.json` to prevent impersonation or untrusted updates.
- **Secure Key Storage**: Manages Nostr keypairs in `~/.envo/keys.json` with owner-only filesystem permissions (`0600`/`0700` on Unix).

---

## Installation

### Unix (Linux & macOS)

Install using the shell installer:

```sh
curl -fsSL https://raw.githubusercontent.com/kaihere14/climenv/main/install.sh | sh
```

*Optional Environment Variables:*
- `ENVO_VERSION`: Specific release tag to install (defaults to `latest`).
- `ENVO_INSTALL_DIR`: Target installation directory (defaults to `$HOME/.local/bin`).

### Windows

Install using PowerShell:

```powershell
irm https://raw.githubusercontent.com/kaihere14/climenv/main/install.ps1 | iex
```

### From Source

If you have Rust and `cargo` installed:

```sh
cargo install --git https://github.com/kaihere14/climenv
```

---

## Getting Started

### 1. Generate an Identity

Before pushing or pulling secrets, create or inspect your local Nostr identity:

```sh
envo keygen
```

This creates a keypair saved at `~/.envo/keys.json` if one does not already exist, and prints your public key (`npub...`). Share your public key with team members so they can authorize you in their `.env-share` files.

---

## Usage

### Pushing Secrets

To publish an environment file, navigate to your project directory. `envo push` requires two files:

1. `.env`: The secret environment key-value pairs you want to share.
2. `.env-share`: A file containing a comma-separated list of recipient public keys (`npub...` or hex) allowed to decrypt the secrets.

Example `.env-share`:
```text
npub1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx,
npub1yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

Publish the secrets under a specific tag:

```sh
envo push <tag>
```

Example:
```sh
envo push my-project-staging
```

`envo` encrypts the `.env` content individually for each key listed in `.env-share` (as well as your own key) and broadcasts the encrypted payload to Nostr relays.

---

### Pulling Secrets

To fetch and decrypt secrets published under a tag:

```sh
envo pull <tag> --owner <owner-npub>
```

- `--owner <owner-npub>`: The public key (`npub...`) of the trusted publisher for this tag. **Required on the first pull** for a tag to pin the trusted owner.

Example:
```sh
envo pull my-project-staging --owner npub1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Once an owner is pinned for a tag, subsequent pulls for that tag no longer require the `--owner` argument:

```sh
envo pull my-project-staging
```

When pulled successfully, `envo` decrypts the content and writes it directly to `.env` in your current working directory.

---

## Project & Storage Structure

### Project Files

- `.env`: Source secret file containing line-separated key-value pairs.
- `.env-share`: Comma-separated list of authorized Nostr public keys (`npub...`).

### Global Configuration (`~/.envo/`)

- `~/.envo/keys.json`: Stores your Nostr keypair (`npub` and `nsec`).
- `~/.envo/trusted_owners.json`: Maps tag names to pinned owner public keys (`{ "tag": "npub..." }`).

---

## Default Relays

`envo` publishes and fetches events using the following Nostr relays:

- `wss://relay.damus.io`
- `wss://nos.lol`
- `wss://purplepag.es`
- `wss://relay.primal.net`
- `wss://relay.nostr.band`