# Monad Testnet Validator — TrieDB Reconfiguration Guide

This guide covers the complete procedure for reconfiguring the TrieDB drive from scratch on an active Monad testnet validator node.

## When to Use

- Corruption or data inconsistency detected on the TrieDB drive
- Node needs to be resynced from a new forkpoint
- Monad team requests a TrieDB reconfiguration

## Prerequisites

- Root access to the server
- `monad` user exists with a valid `.env` file in the home directory
- A dedicated NVMe drive for TrieDB (the `/dev/triedb` symlink must be defined)
- Internet connectivity (required to download the forkpoint and validators.toml)

## Important Warnings

> **Formatting the wrong drive will destroy your operating system.**
> Always complete the drive detection step (Step 2) and verify the `TRIEDB_DEV`
> variable before proceeding.

- All commands must be executed in order — do not skip any step
- `systemctl` commands require root privileges (switch with `sudo -i`)
- The `monad-mpt` and forkpoint steps must be run as the `monad` user
- Never proceed to disk operations before all services are stopped

## Estimated Duration

Depending on forkpoint download speed: **30 minutes to several hours**.

## Reference

Monad official documentation: [docs.monad.xyz](https://docs.monad.xyz)
