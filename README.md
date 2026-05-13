# Setup Bastion SSH

[![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-Setup%20Bastion%20SSH-blue?logo=github)](https://github.com/marketplace/actions/setup-bastion-ssh)
[![License](https://img.shields.io/badge/license-Apache%202.0-green)](LICENSE)

Configure SSH access to private servers through a bastion/jump host inside GitHub Actions workflows.

This action creates an SSH configuration with `ProxyJump` support and allows secure access to internal/private infrastructure from GitHub-hosted runners.

---

## Features

  - Supports bastion/jump host SSH access
  - Supports custom SSH ports
  - Works with private/internal servers
  - Supports configurable strict host checking
  - Supports secure known-host verification mode
  - Simple reusable composite GitHub Action
  - Compatible with GitHub-hosted runners

---

# Quick Start

## Simple mode

Use this mode when convenience is preferred and strict SSH host verification is not required.

```yaml
name: Deploy Application

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout your code
        uses: actions/checkout@v6

      - name: Setup SSH
        uses: rishav2404/setup-bastion-ssh@v1
        with:
          ssh-private-key: ${{ secrets.SSH_KEY }}

          bastion-host: ${{ secrets.BASTION_HOST }}
          bastion-user: ${{ secrets.BASTION_USER }}
          bastion-port: "9222"

          server-host: ${{ secrets.SERVER_HOST }}
          server-user: ${{ secrets.SERVER_USER }}

      - name: Perform SSH action
        run: |
          ssh target 'cd /xyz && ./deploy.sh'
```

---

# Inputs

| Name                   | Required | Default | Description                                                |
| ---------------------- | -------: | ------: | ---------------------------------------------------------- |
| `ssh-private-key`      |      Yes |       - | SSH private key used for authentication                    |
| `bastion-host`         |      Yes |       - | Bastion/jump host IP or hostname                           |
| `bastion-user`         |      Yes |       - | SSH username for bastion host                              |
| `bastion-port`         |       No |    `22` | SSH port for bastion host                                  |
| `bastion-known-host`   |       No |       - | Known-host line for bastion host, required in secure mode  |
| `server-host`          |      Yes |       - | Target/internal server IP or hostname                      |
| `server-user`          |      Yes |       - | SSH username for target server                             |
| `server-port`          |       No |    `22` | SSH port for target server                                 |
| `server-known-host`    |       No |       - | Known-host line for target server, required in secure mode |
| `strict-host-checking` |       No |    `no` | SSH `StrictHostKeyChecking` option                         |

---

# SSH Key Setup

Generate an SSH key pair:

```bash
ssh-keygen -t ed25519
```

Add the public key to:

```text
~/.ssh/authorized_keys
```

on:

- bastion host
- target server

Store the private key inside GitHub Secrets:

```text
SSH_KEY
```

---

# Secure Mode

If you set:

```yaml
strict-host-checking: "yes"
```

then you must also provide:

- `bastion-known-host`
- `server-known-host`

These values are written into `known_hosts` before the SSH connection is attempted.

This enables proper SSH host verification and protects against man-in-the-middle attacks.

---

# Generating Known Hosts

## Bastion host

Run this from your local machine:

```bash
ssh-keyscan -p <BASTION_PORT> <BASTION_HOST>
```

```bash
Example:
ssh-keyscan -p 9222 bastion.example.com
OR
ssh-keyscan -p 4222 15.xx.xx.xx
```

Output:

```text
[14.139.240.89]:9222 ssh-ed25519 ABZZC8NlmC4jHJL1NTE5BBBA...
```

Store this line as GitHub Secret:

```text
BASTION_KNOWN_HOST
```

---

## Target server

Run this from the bastion host or from a machine that can access the target server:

```bash
ssh-keyscan -p <SERVER_PORT> <SERVER_HOST>
```

Example:

```bash
ssh-keyscan -p 22 172.xx.xx.108
```

Output:

```text
172.xx.xx.108 ssh-ed25519 ABZZC8NlmC4jHJL1NTE5BBBA...
```

Store this line as GitHub Secret:

```text
SERVER_KNOWN_HOST
```

---

# Secure Mode Example

```yaml
name: Deploy Application

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout your code
        uses: actions/checkout@v6

      - name: Setup SSH
        uses: rishav2404/setup-bastion-ssh@v1
        with:
          ssh-private-key: ${{ secrets.SSH_KEY }}

          bastion-host: ${{ secrets.BASTION_HOST }}
          bastion-user: ${{ secrets.BASTION_USER }}
          bastion-port: "9222"
          bastion-known-host: ${{ secrets.BASTION_KNOWN_HOST }}

          server-host: ${{ secrets.SERVER_HOST }}
          server-user: ${{ secrets.SERVER_USER }}
          server-known-host: ${{ secrets.SERVER_KNOWN_HOST }}

          strict-host-checking: "yes"

      - name: Perform SSH action
        run: |
          ssh target 'cd /xyz && ./deploy.sh'
```

---

# Example Infrastructure

```text
GitHub Actions Runner
        |
        v
  Bastion Host
        |
        v
 Private Server
```

---

# Security Notes

By default:

```text
StrictHostKeyChecking=no
```

is used for convenience and compatibility with ephemeral CI runners.

For production environments, enable:

```yaml
strict-host-checking: "yes"
```

and provide:

- `bastion-known-host`
- `server-known-host`

This enables proper SSH host verification.

---

# Troubleshooting

## Permission denied (publickey)

Ensure:

- the correct SSH private key is stored in `SSH_KEY`
- the corresponding public key exists in:

  ```text
  ~/.ssh/authorized_keys
  ```

  on:
  - bastion host
  - target server

---

## Host key verification failed

If using:

```yaml
strict-host-checking: "yes"
```

ensure:

- `bastion-known-host` is provided
- `server-known-host` is provided
- fingerprints are correct

---

## Connection timeout

Ensure:

- bastion host is reachable from the internet
- SSH port is open
- target server is reachable from bastion

---

# Requirements

- SSH access enabled on bastion and target hosts
- Bastion host reachable from the internet
- Target server reachable from the bastion
- OpenSSH installed on target systems

---

# License

Apache 2.0
