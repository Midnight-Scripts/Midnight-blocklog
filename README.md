# Midnight-Blocklog

Aura schedule + block production logger for a Midnight (Substrate) node.

Features:
- Computes your Aura slots for the current epoch (session) and stores them in SQLite
- Watches the chain and updates slot status:
  - `schedule` (planned)
  - `mint` (seen on best head)
  - `finality` (seen finalized)
- Stores per-epoch authority set metadata
- Timezone conversion + optional color output

## Requirements

- A running `midnight-node` with WS RPC enabled (e.g. `--unsafe-rpc-external` `--rpc-methods=Unsafe` `--rpc-port 9944`)

## Build dependencies (Linux)

`mblog` uses `rusqlite` with the `bundled` feature, so you do not need a system `libsqlite3`. You do need a C toolchain.

Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y build-essential pkg-config libssl-dev
```

## Install Rust (rustup)

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup toolchain install stable
rustup default stable
rustc -V
cargo -V
```


## Build

```bash
cargo build --bin mblog
```

## Run

```bash
./target/debug/mblog \
  --keystore-path /path/to/your/keystore \
  --watch \
  --tz Asia/Dubai \
  --db aura_schedule.sqlite
```

Common options:
- `--ws ws://...` (Midnight node WS RPC endpoint; default: ws://127.0.0.1:9944)
- `--keystore-path` (/path/to/keystore: required)
- `--epoch-size N` (number of slots per epoch; default: 1200)
- `--watch` (enable chain watching mode; default: off)
- `--tz TIMEZONE` (timezone for output times; default: UTC) Timezone fomat type `UTC`|`local`|`+09:00`|`-05:00`|`Area/City`
- `--color auto|always|never` (output color mode; default: auto)
- `--no-store` (print only; no SQLite writes)

## SQLite schema

### Epoch info (`epoch_info`)
- `epoch`
- `start_slot`
- `end_slot`
- `authority_set_hash`
- `authority_set_len`
- `created_at_utc`

### Block slots (`blocks`)
- `slot` (primary key)
- `epoch`
- `planned_time_utc`
- `block_number` (nullable)
- `block_hash` (nullable)
- `produced_time_utc` (nullable)
- `status` (`schedule` | `mint` | `finality`)

## Security

- This tool never reads or prints secret keys.
- `author_hasKey` is a local-node check: it confirms the keystore contains the secret key for the detected public key.

---

# Midnight-Blocklog（日本語）

Midnight（Substrate）ノード向けの **Aura スケジュール算出 + ブロック生成ログ（SQLite 記録）** ツールです。

主な機能:
- 現在エポック（セッション）の **自分の担当スロット**を計算し、SQLite に `schedule` として記録
- チェーンを監視し、スロットの状態を更新:
  - `schedule`（予定）
  - `mint`（best head で観測）
  - `finality`（finalized で観測）
- エポックごとの authority set 情報を SQLite に記録
- タイムゾーン変換、色付き表示に対応

## 必要条件

- `midnight-node` が起動しており、WS RPC が有効（例: `--rpc-port 9944`）
- `midnight-node` 起動時に必要なオプション `--unsafe-rpc-external` `--rpc-methods=Unsafe`

## ビルド

```bash
cargo build --bin mblog
```

## 実行

```bash
./target/debug/mblog \
  --keystore-path /path/to/your/keystore \
  --watch \
  --tz Asia/Dubai \
  --db aura_schedule.sqlite
```

主なオプション:
- `--ws ws://...`（Midnight ノードの WS RPC エンドポイント、デフォルト: ws://127.0.0.1:9944）
- `--keystore-path /path/to/keystore`（keystore のパス、必須）
- `--epoch-size N`（エポックあたりのスロット数、デフォルト: 1200）
- `--watch`（チェーン監視モードを有効にする、デフォルト: 無効）
- `--tz TIMEZONE` (timezone for output times; default: UTC)  タイムゾーン形式タイプ `UTC`|`local`|`+09:00`|`-05:00`|`Area/City`
- `--color auto|always|never` （出力の色付きモード、デフォルト: auto）
- `--no-store`（SQLite記録無効。表示のみ）

## SQLite スキーマ

### エポック情報（`epoch_info`）
- `epoch`（エポック番号）
- `start_slot`（開始スロット）
- `end_slot`（終了スロット）
- `authority_set_hash`
- `authority_set_len`
- `created_at_utc`

### ブロック情報（`blocks`）
- `slot`（主キー）
- `epoch`
- `planned_time_utc`（予定時刻）
- `block_number`（任意）
- `block_hash`（任意）
- `produced_time_utc`（任意）
- `status`（`schedule` | `mint` | `finality`）

## 補足 / 注意書き
- 起動時に `author_hasKey` を確認し、このノードの keystore に秘密鍵が無い場合は起動しません。

## セキュリティ

- 秘密鍵は読み取りません・表示しません。
- `author_hasKey` は「ローカルノードが秘密鍵を持っているか」の確認です。
