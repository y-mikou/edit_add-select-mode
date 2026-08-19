# ![Application Icon for Edit](./assets/edit.svg) Edit

A simple editor for simple needs.

This editor pays homage to the classic [MS-DOS Editor](https://en.wikipedia.org/wiki/MS-DOS_Editor), but with a modern interface and input controls similar to VS Code. The goal is to provide an accessible editor that even users largely unfamiliar with terminals can easily use.

![Screenshot of Edit with the About dialog in the foreground](./assets/edit_hero_image.png)

## Installation

[![Packaging status](https://repology.org/badge/vertical-allrepos/microsoft-edit.svg?exclude_unsupported=1)](https://repology.org/project/microsoft-edit/versions)

You can also download binaries from [our Releases page](https://github.com/microsoft/edit/releases/latest).

### Windows

You can install the latest version with WinGet:
```powershell
winget install Microsoft.Edit
```

### Linux (build from source)

If your distribution does not provide binaries, or if you'd like to build your own, you can use our install script, provided you have installed:
* Rust (via `rustup` or similar)
* A C compiler (e.g. `gcc`)
* ICU (e.g. libicu78, libicu, icu)
* curl/wget and tar

The following command will then install `msedit` into `~/.local/bin`:
```sh
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/microsoft/edit/main/assets/install.sh | sh
```

Additional flags are `--dev`, to build directly from the main branch, and `--system` to install into `/usr/local/bin`. For instance:
```sh
curl --proto '=https' --tlsv1.2 -sSf https://raw.githubusercontent.com/microsoft/edit/main/assets/install.sh | sh -s -- --dev --system
```

### macOS

You can install the latest version with Homebrew:
```sh
brew install msedit
```

## Build Instructions

* [Install Rust](https://www.rust-lang.org/tools/install)
* Clone the repository
* If you're using nightly Rust:
  ```sh
  cargo build --release --config .cargo/release.toml
  ```
* If you're using stable Rust:
  * Ideally: Set the environment variable `RUSTC_BOOTSTRAP=1` and use the **nightly** build instructions above.
    This is recommended, because it drastically reduces the binary size and slightly improves performance.
  * Otherwise, simply run:
    ```sh
    cargo build --release
    ```

### Build Configuration

You can set the following environment variables at build-time to configure the build:

Environment variable | Description
--- | ---
`EDIT_CFG_ICU*` | See [ICU library name (SONAME)](#icu-library-name-soname) below for details. Linux package maintainers are advised to review and configure these options.
`EDIT_CFG_LANGUAGES` | A comma-separated list of languages to include in the build. See [i18n/edit.toml](i18n/edit.toml) for available languages.

## 範囲選択モード(本家Editに対する追加機能)

- 概要: カーソル位置を「選択始点」として記憶し、モード中はテキスト入力を破棄してカーソル移動のみを受け付けます。モードを終了すると現在のカーソル位置が「選択終点」となり、始点から終点までが選択状態になります。これは、ターミナルと入力メソッドの組み合わせによってshift+↑やshift+↓での行をまたいだ選択範囲の拡大縮小が機能しないことへの対応です。
- 使い方:
  - メニューバーの `Edit` → `範囲選択の開始/終了` をクリックしてトグルします。
  - ショートカット: `Alt+R`（メニューを開かず直接トグルできます）。
- 注意: モード中はカーソル移動・メニュー操作・ウィンドウリサイズ等のみが処理され、通常の文字入力やペーストは破棄されます。
- 注意: この機能は日本語での入力、linux(debian)での動作のみをスコープとしており他の状況での考慮をしていません。

## リリースビルド（正式ビルド）

このリポジトリでは `crates/edit` にてリリース用バイナリ `edit_r-sel` を明示的に定義しています。正式な（最適化された）ビルド手順と基本的なインストール例は以下の通りです。

- Release ビルド（ワークスペースのルートで実行）:
  ```bash
  cargo build --package edit --release --bin edit_r-sel
  ```

- 出力バイナリ:
  - 成功するとバイナリは `target/release/edit_r-sel` に作成されます。

- 実行例:
  ```bash
  ./target/release/edit_r-sel <file-to-open>
  ```

- システムにインストールする例（管理者権限が必要）:
  ```bash
  sudo cp target/release/edit_r-sel /usr/local/bin/edit_r-sel
  ```

- ビルド時の環境変数:
  - ICU の SONAME や組み込む言語の設定は既存の `Build Configuration` セクションで触れている `EDIT_CFG_*` 環境変数を参照してください。必要に応じて `EDIT_CFG_ICUUC_SONAME` や `EDIT_CFG_ICUI18N_SONAME` を設定してビルドしてください。

注: `crates/edit/Cargo.toml` で `autobins = false` を設定しているため、`edit_r-sel` を明示的に `--bin` 指定してビルドするか、上記のコマンドを使用してください。

## Notes to Package Maintainers

### Package Naming

The canonical executable name is "edit" and the alternative name is "msedit".
We're aware of the potential conflict of "edit" with existing commands and recommend alternatively naming packages and executables "msedit".
Names such as "ms-edit" should be avoided.
Assigning an "edit" alias is recommended, if possible.

### ICU library name (SONAME)

This project optionally depends on the ICU library for its Search and Replace functionality.

By default, the project will look for the following library names:

 Variable | Windows | macOS | Linux / Other
----------|---------|-------|---------------
`EDIT_CFG_ICUUC_SONAME` | `icuuc.dll` | `libicucore.dylib` | `libicuuc.so`
`EDIT_CFG_ICUI18N_SONAME` | `icuin.dll` | `libicucore.dylib` | `libicui18n.so`

If your installation uses a different SONAME, please set the following environment variable at build time:
* `EDIT_CFG_ICUUC_SONAME`:
  For instance, `libicuuc.so.76`.
* `EDIT_CFG_ICUI18N_SONAME`:
  For instance, `libicui18n.so.76`.

Additionally, this project assumes that the ICU exports symbols without `_` prefix and without version suffix, such as `u_errorName`.
If your installation uses versioned exports, please set:
* `EDIT_CFG_ICU_CPP_EXPORTS`:
  If set to `true`, it'll look for C++ symbols such as `_u_errorName`.
  Enabled by default on macOS.
* `EDIT_CFG_ICU_RENAMING_VERSION`:
  If set to a version number, such as `76`, it'll look for symbols such as `u_errorName_76`.

Finally, you can set the following environment variables:
* `EDIT_CFG_ICU_RENAMING_AUTO_DETECT`:
  If set to `true`, the executable will try to detect the `EDIT_CFG_ICU_RENAMING_VERSION` value at runtime.
  The way it does this is not officially supported by ICU and as such is not recommended to be relied upon.
  Enabled by default on UNIX (excluding macOS) if no other options are set.

To test your build settings, run `cargo test` with the `--ignored` flag. For instance:
```sh
cargo test -- --ignored
```
