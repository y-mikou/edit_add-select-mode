# Edit(範囲選択モード追加)

基本機能は本家のEdit(https://github.com/microsoft/edit)に準拠します。

当リポジトリへフォークされたのは2026/08/19です(このバージョンは`org_edit`ブランチに保持されています)。

本家Editと異なるのは、以下の点です。

## 範囲選択モード(本家Editに対する追加機能)

- 概要: カーソル位置を「選択始点」として記憶し、モード中はテキスト入力を破棄してカーソル移動のみを受け付けます。モードを終了すると現在のカーソル位置が「選択終点」となり、始点から終点までが選択状態になります。これは、ターミナルと入力メソッドの組み合わせによってshift+↑やshift+↓での行をまたいだ選択範囲の拡大縮小が機能しないことへの対応です。
- 使い方:
  - メニューバーの `Edit` → `範囲選択の開始/終了` をクリックしてトグルします。
  - ショートカット: `Alt+R`（メニューを開かず直接トグルできます）。
- 注意: モード中はカーソル移動・メニュー操作・ウィンドウリサイズ等のみが処理され、通常の文字入力やペーストは破棄されます。
- 注意: この機能は日本語での入力、linux(debian)での動作のみをスコープとしており他の状況での考慮をしていません。

## ビルド

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
