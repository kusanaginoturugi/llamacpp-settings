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
