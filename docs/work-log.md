# 作業記録

## 2026-07-28

- `/etc/llama.cpp/models.ini` と `llama.cpp.service` の実設定を確認した。
- `README.md` を作成し、モデル設定、systemd user service、反映手順、API 疎通確認を記載した。
- `/home/onoue/.config/systemd/user/llama.cpp.service` の `--path` 行に継続用の `\` を追加した。
- `/home/onoue/.local/lib/models/` から chat-template 4 件を `models/` にコピーした。
- `models/` のテンプレートとコピー元の SHA256 が一致することを確認した。
- README に AUR `llama.cpp-cuda`、GPU/CUDA 確認、linger、OpenAI 互換 API、Hugging Face キャッシュ、VRAM 調整指針を追記した。
- README から不要な `ExecStart` 継続行の注意書きを削除した。
- `/home/onoue/src/oss/llama.cpp/tools/ui/dist` を参考用に `ui-dist/` へコピーした。
- README に `--path /home/onoue/src/oss/llama.cpp/tools/ui/dist` が日本語化済み Web UI dist を指定するためのものだと追記した。
- 日本語化された Web UI 設定画面のスクリーンショットを `docs/images/llama-ui-settings-ja.png` に追加した。
- README の `models.ini` 説明を修正し、Hugging Face 取得の `hf` / `hf-repo` / `hf-file` を使う前提に寄せた。
- README から手動ダウンロードした GGUF をローカルファイル指定で使う説明を削除した。
- README に新しいモデルを `models.ini` へ追加する前の検証手順を追記した。

## 確認事項

- `llama serve --help` で `-hf`、`HF_TOKEN`、`--no-webui`、`--cache-list` が使えることを確認した。
- この作業環境では `nvidia-smi` が NVIDIA driver に接続できず、`llama-server --version` でも CUDA 初期化エラーが出た。
- service の再起動や API 実リクエストは未実施。
- `ui-dist/` は 67 ファイル、約 8.8MB。
- `docs/images/llama-ui-settings-ja.png` は 1242x1181 の PNG。
- OpenAI 互換 API の JSON にある `"model"` は `/v1/models` で見えるモデル ID を指定するものとして記載した。
- `llama-cli` が `/usr/bin/llama-cli` に存在することを確認した。

## 2026-07-31

- README の `models.ini` 例を現行設定に合わせて更新した。
- README から通常運用での `chat-template-file` 指定を削除した。
- GGUF 内蔵の chat template を使うため、通常は `chat-template-file` を書かなくてよいことを追記した。
- `ui = true` / `ui = false` と `[*] ui = false` の使い分けを追記した。
- モデル追加手順の INI 例から `chat-template-file` を削除した。
- README の systemd service 例に `--cors-origins` と `--ui-mcp-proxy` を追加した。
- `--cors-origins` は Offline-llm-translator ブラウザ拡張機能用、`--ui-mcp-proxy` は Web UI から MCP を使うための設定として説明した。
- README に `ihor-sokoliuk/mcp-searxng` を HTTP mode で起動し、llama.cpp Web UI の MCP 設定へ追加する手順を追記した。
- `mcp-searxng` 用の systemd user service 例と、SearXNG JSON API / `/health` の確認手順を追記した。
- README の MCP 節を整理し、`usememos/memos` の組み込み MCP server を llama.cpp Web UI へ登録する手順を追記した。
- Memos MCP 設定画面のスクリーンショットを `docs/images/memos-mcp-config.png` に追加した。
- llama.cpp のビルトインツール一覧を確認し、README に `write_file` / `edit_file` / `exec_shell_command` の権限リスクを追記した。
- `write_file` / `edit_file` は `llama-server` の実行ユーザー権限で動き、専用 sandbox や書き込み先制限はないことを確認した。
- README の MCP 章に、後で試す候補として fetch / URL reader、filesystem read-only、git read-only、memory、sqlite、Playwright、MarkItDown 系を追記した。
- MCP server を追加する際は HTTP 系 transport、read-only、ローカル待ち受け、必要なディレクトリだけ許可する方針を追記した。

## 2026-11-16

- `llama.cpp.service` の `ExecStart` が、パッケージ版の `/usr/bin/llama-server` からビルド版の `/home/onoue/src/oss/llama.cpp-runtime/build/bin/llama-server` に変わっていたので、README を合わせた。パッケージ版は更新が遅いのが理由。
- README のインストール手順を AUR `llama.cpp-cuda` からソースビルドに書き換えた。ビルド設定は `build/CMakeCache.txt` から拾った（v0.9.0、Release、`GGML_CUDA=ON`、`CMAKE_CUDA_ARCHITECTURES=86`、Ninja）。
- 更新手順（タグを checkout、ビルドし直し、サービス再起動）を追記した。
- `llama-cpp` 0.9.0-1.1 パッケージの `/usr/bin/llama-server` と `/usr/bin/llama-cli` がまだ残っていて、PATH 上はそちらが先に見つかる。そのため README の `llama-server` 実行例はビルド版のフルパスにした。`llama-cli` はビルドしていないのでパッケージ版のまま。
- README と handoff の systemd 例・説明を実際の `llama.cpp.service` に合わせた。`--models-max` を 1 から 2 に、`--tools` から `get_datetime` を外し、`--cors-origins` の拡張機能 ID を `3b7be389-…` に変更。
- v0.9.0 のビルトインツールに `get_datetime` がなく、代わりに `get_info`（OS 名・バージョンと cwd）があるので、README の表を更新した。
- chat-template の説明を「通常は書かない」から実態に合わせ、`chat-template-file` を指定している 7 プリセットの一覧を README に載せた。
- work-plan の `/etc/llama.cpp/models.ini` を `~/.config/llama.cpp/models.ini` に直した。
- `--models-max 2` でも載る組み合わせは VRAM 次第だと README に追記した。E4B と plamo-3-translate は同時に載るが、12B は E4B と同時に載らない（`cudaMalloc failed: out of memory` で 500 になる）。
- handoff の「CUDA が見えていない」という古い注意を、GPU 動作確認済みに更新した。
- `models.ini` の `[Gemma-4-E4B-Uncensored]` にあった `mmproj=` の絶対パスを削除した。片方の PC の snapshot ハッシュを指していて、もう片方の PC では存在しなかった。`hf` 指定なので、mmproj は llama-server が自動で取得する（HF リポジトリに `mmproj-…-f16.gguf` があることを確認）。
- もう 1 台の PC のコミット（service のリポジトリ化、llama.env、VRAM 実測、vntl 削除）と合流した。models.ini は VRAM 実測の値を土台に、こちらで足した plamo・hy-mt1.5・lfm2.5・bge-m3 を加えた。こちらで消していた gemma-4-12B-it-qat と translategemma-4b は、向こうで設定を詰めていたので残した。
- `systemd/llama.cpp.service` の `ExecStart` を、パッケージ版の `/usr/bin/llama-server` からビルド版の `%h/src/oss/llama.cpp-runtime/build/bin/llama-server` に変えた。どの PC も同じ場所にビルドしておく前提。
- README の systemd の節を、リポジトリの service と `llama.env` を前提にした説明に直し、chat-template の表から消えたプリセット（vntl、translategemma-12b）を外した。
- この PC で `./deploy -f` を実行した。`~/.config/llama.cpp/models.ini` はリンクではなく 11/5 時点のコピーだった（中身は合流前のローカル版と同一だったので失うものはなし）。元のファイルは `models.ini.bak` と `llama.cpp.service.bak` に残っている。`llama.env` は旧 service の `--cors-origins` から作られた。再起動後、ビルド版で起動し、拡張のオリジンへの CORS 許可と翻訳 API の応答を確認した。
