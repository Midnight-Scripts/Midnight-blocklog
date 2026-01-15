# Midnight-blocklog (English)

A tool for Midnight nodes that **displays the Aura block production schedule and records it in SQLite**.

※This tool is currently in beta. Specifications may change and backward-incompatible changes may occur before the official release.

This tool **auto-detects the Aura public key** from the node keystore, verifies that **this node holds the corresponding secret key** via `author_hasKey`, then calculates and records the assigned slots for the current session (referred to as “epoch” here for convenience).

## What it does

- Calculates your **assigned Aura slots** in the current epoch (session), displays them, and stores them in SQLite as `schedule`
- In watch mode (`mblog block --watch`), tracks the chain and updates the status. It waits until the next session, and at the boundary it calculates and stores the assigned slots for the new epoch.
  - `schedule` (planned)
  - `mint` (observed on best head)
  - `finality` (observed on finalized)
- Stores Authority set information per epoch (hash/length, start/end slots, etc.)
- Supports output timezone selection and colored output (auto-detected via TTY)

## Requirements

- `midnight-node` must be started with the following flags (WS RPC enabled
  `--rpc-methods=Unsafe`
  `--unsafe-rpc-external`
  `--rpc-port 9944`
- Rust (`cargo`) build environment

## Install Rust (rustup)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup toolchain install stable
rustup default stable
rustc -V
cargo -V
```

## Build dependencies (Linux)
Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y build-essential pkg-config libssl-dev
```

## Install (clone this repository and run `cargo install`)

```bash
git clone https://github.com/Midnight-Scripts/Midnight-blocklog.git
cd Midnight-blocklog
git checkout <latest_tag_name>
cargo install --path . --bin mblog --locked --force
```

`mblog` is typically installed to `~/.cargo/bin/mblog`.

## Usage

### 1) Show help

```bash
mblog --help
```

Output (actual `--help`):

```text
Usage: mblog <COMMAND>

Commands:
  block  Show Aura slot schedule (use --watch to monitor)
  log    Show stored blocks from SQLite
```

## Options

Options are provided per subcommand.

### `mblog block`

- `--ws <WS>`: WS RPC endpoint (optional; default: `ws://127.0.0.1:9944`)
- `--keystore-path <KEYSTORE_PATH>`: Node keystore directory (required)
- `--epoch-size <EPOCH_SIZE>`: Number of slots per epoch (optional; default: `1200`)
- `--lang <LANG>`: Language for fixed messages (optional; `ja` | `en`; default: `en`)
- `--tz <TZ>`: Output timezone (optional; default: `UTC`)
  - `UTC` / `local` / `+HH:MM` / `-HH:MM`
  - Unix only: IANA timezones such as `Asia/Tokyo` (sets `TZ` internally and uses system tzdata)
- `--color <auto|always|never>`: Colored output (optional; default: `auto`)
- `--db <DB>`: SQLite DB path (optional; default: `./mblog.db`)
- `--no-store`: Do not write to SQLite (optional; logs only; `--db` path is not required)
- `--ariadne-endpoint <ARIADNE_ENDPOINT>`: Ariadne JSON-RPC endpoint used for sidechain registration checks (optional; default: `https://rpc.testnet-02.midnight.network`)
- `--ariadne-insecure`: Accept invalid TLS certs for Ariadne endpoint (optional)
- `--no-registration-check`: Disable sidechain registration check (optional)
- `--watch`: Continuous monitoring (optional; keeps running without exiting)
- `--output-json`: Output schedule JSON to stdout (optional; cannot be used with `--watch`; exits after printing)
- `--current`: Output the current epoch schedule (requires `--output-json`)
- `--next`: Output the next epoch schedule (requires `--output-json`)

### `mblog log`

- `--db <DB>`: SQLite DB path (optional; default: `./mblog.db`)
- `--epoch <EPOCH>`: Epoch number to display (optional; default: latest)
- `--tz <TZ>`: Scheduled time timezone (optional; default: `UTC`)

See `mblog block --help` and `mblog log --help` for the authoritative list.


### 2) Schedule DB Save, Display Time Zone, Enable Monitoring Mode

```bash
mblog block \
  --keystore-path /path/to/your/keystore \
  --db /path/to/midnight-dir/mblog.db \
  --tz Asia/Tokyo \
  --watch
```

Example results (may vary depending on time zone settings)
```
 Midnight-blocklog - Version: 0.3.1
--------------------------------------------------------------
epoch:245527 (start_slot:294632400 / end_slot:294633599)
      author: 0x52cc8d7dbb573b0fa3ba8e12545affa48313c3e5e0dc0b07515fd52419373360
   ADA Stake: 2816841.654532 ADA (2816841654532 lovelace)
Registration: true (Registered)

Your Block Schedule List
-------------------------
#1 slot 294633422: 2026-01-07T19:42:12+04:00 (UTC 2026-01-07T15:42:12+00:00)
Total=1

Waiting for next session... (next_epoch=245528)
progress [============================= ] 99% (slot 294633599/294633599)
```
>If there is no schedule, it will display `No schedule for this session`.


### 3) JSON schedule output (stdout)

When `--output-json` is set (instead of `--watch`), `mblog` prints the schedule as JSON to stdout (`date` respects `--tz`) and exits.  
Note: `--output-json` does not write to SQLite (it ignores `--db` and does not create/update the DB).

Examples:

```bash
# Current epoch schedule as JSON (date respects --tz)
mblog block --keystore-path /path/to/keystore --tz UTC --output-json --current

# Next epoch schedule as JSON (date respects --tz)
mblog block --keystore-path /path/to/keystore --tz UTC --output-json --next
```

Sample output:

```json
{
  "epoch": 245555,
  "schedule": [
    { "slot": 294663162, "date": "2026-01-10T12:34:56Z" }
  ]
}
```

### 4) Show stored blocks (SQLite)

```bash
# Latest epoch (default)
mblog log --db /path/to/midnight-dir/mblog.db

# Specific epoch
mblog log --db /path/to/midnight-dir/mblog.db --epoch 245525
```

Example results (may vary depending on time zone settings)
```
Midnight Block Log
-------------------

epoch: 245528
|===|==========|==============|===========|===============|===========================|=======================|
| # | status   | block_number | slot      | slot_in_epoch | Scheduled_time            | block_hash            |
|===|==========|==============|===========|===============|===========================|=======================|
| 1 | finality | 3238956      | 294633833 | 233           | 2026-01-07T20:23:18+04:00 | 0xec7a91ac...81f5d053 |
| 2 | finality | 3238966      | 294633843 | 243           | 2026-01-07T20:24:18+04:00 | 0x63ec2189...c0776574 |
|===|==========|==============|===========|===============|===========================|=======================|
```


## What is stored in SQLite
The data stored in SQLite is continuously updated by running this application with `mblog watch`.

On the first run, an SQLite database is created at the `--db` path you specify, and data is accumulated in the following tables. Please note that if you change the path or omit it, a new database will be created.

### Epoch info (`epoch_info`)

- `epoch`: Epoch number
- `start_slot`: Start slot
- `end_slot`: End slot
- `authority_set_hash`: Hash of the Authority set
- `authority_set_len`: Number of elements in the Authority set
- `created_at_utc`: Recorded time (UTC)

### Block info (`blocks`)

- `slot` (primary key)
- `epoch`
- `planned_time_utc`: Planned block production time (UTC)
- `block_number`
- `block_hash`
- `produced_time_utc`
- `status`: `schedule` / `mint` / `finality`

## Security

- This tool does not read or print secret keys (it detects the public key from keystore filenames).
- `author_hasKey` is an RPC that checks whether this node’s keystore contains the corresponding secret key.


## Roadmap
- Indexer Integration
- UX improvements (please open an issue if you have a request)

## License
Apache-2.0

Copyright (c) 2026 BTBF (X-StakePool)


Copyright (c) 2026 BTBF (X-StakePool)

---
