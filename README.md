# Midnight-blocklog

Midnightノード向けの **Aura ブロック生成スケジュール表示 + SQLite 記録** ツールです。

このツールは、ノードの keystore から **Aura 公開鍵を自動検出**し、`author_hasKey` で **このノードが秘密鍵を保持していること**を確認したうえで、現在セッション（ここでは便宜上「epoch」と表記）の担当スロットを計算して記録します。

## できること

- 現在 epoch（session）の **自分の Aura 担当スロット**を計算して表示・SQLite に `schedule` として保存
- 監視モード（`--watch`）でチェーンを追跡し状態を更新。次のセッションまで待機し境界で新しい epoch の担当スロットを計算・保存します。
  - `schedule`（予定）
  - `mint`（best head で観測）
  - `finality`（finalized で観測）
- epoch ごとに Authority セット情報を保存（ハッシュ/長さ、開始/終了スロットなど）
- 出力タイムゾーン指定、色付き出力（TTY 自動判定）

## 動作要件

- `midnight-node`の起動オプションに以下のフラグを追加（WS RPC 有効
  `--rpc-methods=Unsafe`
  `--unsafe-rpc-external`
  `--rpc-port 9944`
- Rust（`cargo`）ビルド環境

## Rust のインストール（rustup）

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
rustup toolchain install stable
rustup default stable
rustc -V
cargo -V
```

## ビルド依存（Linux）
Ubuntu/Debian:

```bash
sudo apt-get update
sudo apt-get install -y build-essential pkg-config libssl-dev
```

## インストール（このリポジトリを clone して `cargo install`）

```bash
git clone https://github.com/btbf/Midnight-blocklog.git
cd Midnight-blocklog
cargo install --path . --bin mblog --locked --force
```

インストール後、`mblog` は通常 `~/.cargo/bin/mblog` に配置されます。

## 使い方

### 1) ヘルプ表示

```bash
mblog --help
```

出力（実際の `--help`）:

```text
Usage: mblog [OPTIONS] --keystore-path <KEYSTORE_PATH>

Options:
      --ws <WS>                        [default: ws://127.0.0.1:9944]
      --keystore-path <KEYSTORE_PATH>  Path to the node's keystore directory. The Aura public key is auto-detected from this
      --epoch-size <EPOCH_SIZE>        [default: 1200]
      --lang <LANG>                    Output language for fixed messages: ja|en [default: en] [possible values: ja, en]
      --tz <TZ>                        Output timezone: "UTC", "local", fixed offset like "+09:00"/"-05:00", or an IANA zone like "Asia/Dubai" (Unix only; uses system tzdata via TZ env) [default: UTC]
      --color <COLOR>                  Colorize output: auto|always|never [default: auto] [possible values: auto, always, never]
      --db <DB>                        SQLite DB path [default: mblog.sqlite]
      --no-store                       Do not write to SQLite
      --watch                          Enable continuous monitoring mode (run forever)
  -h, --help                           Print help
  -V, --version                        Print version
```

### 2) DB保存、表示タイムゾーン、監視モード

`--db`は必ず指定し一度作成したら同じパスを使い続けてください。

```bash
cd /path/to/Midnight-dir
mblog \
  --keystore-path /path/to/your/keystore \
  --db /path/to/midnight-dir/mblog.sqlite \
  --tz Asia/Tokyo \
  --watch
```

## オプション

`--ws` は省略可能です（デフォルト: `ws://127.0.0.1:9944`）。

- `--ws <WS>`: WS RPC エンドポイント（省略可）
- `--keystore-path <KEYSTORE_PATH>`: ノード keystore ディレクトリ（必須）
- `--epoch-size <EPOCH_SIZE>`: 1 epoch あたりのスロット数（デフォルト: `1200`）
- `--lang <LANG>`: 固定メッセージの言語（`ja` | `en`、デフォルト: `en`）
- `--tz <TZ>`: 出力タイムゾーン（デフォルト: `UTC`）
  - `UTC` / `local` / `+HH:MM` / `-HH:MM`
  - Unix のみ: `Asia/Tokyo` のような IANA タイムゾーン（内部で `TZ` を設定し、システムの tzdata を利用）
- `--color <auto|always|never>`: 色付き出力（デフォルト: `auto`）
- `--db <DB>`: SQLite DB パス（必須）
- `--no-store`: SQLite に書き込まない（ログ表示のみ。`--db`パス不要）
- `--watch`: 常時監視（終了せずに動作し続ける）

## SQLite に保存する内容
SQLiteに格納されるデータは、当アプリケーションを`--watch`オプション付きで実行することで継続的に更新されます。

### epoch 情報（`epoch_info`）

- `epoch`: エポック番号
- `start_slot`: 開始スロット
- `end_slot`: 終了スロット
- `authority_set_hash`: Authority セットのハッシュ
- `authority_set_len`: Authority セットの要素数
- `created_at_utc`: 記録時刻（UTC）

### ブロック情報（`blocks`）

- `slot`（主キー）
- `epoch`
- `planned_time_utc`: ブロック生成予定時刻（UTC）
- `block_number`
- `block_hash`
- `produced_time_utc`
- `status`: `schedule` / `mint` / `finality`

## セキュリティ

- このツールは秘密鍵を読み取りません・表示しません（keystore のファイル名から公開鍵を検出します）。
- `author_hasKey` は「このノードの keystore に該当する秘密鍵があるか」を確認する RPC です。


## 今後の予定
- ブロック生成実績一覧の表示機能（エポックごと）
- UX改善（リクエストがあればissueを提出してください）

## ライセンス
Apache-2.0

Copyright (c) 2026 BTBF (X-StakePool)
