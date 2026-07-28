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

## 確認事項

- `llama serve --help` で `-hf`、`HF_TOKEN`、`--no-webui`、`--cache-list` が使えることを確認した。
- この作業環境では `nvidia-smi` が NVIDIA driver に接続できず、`llama-server --version` でも CUDA 初期化エラーが出た。
- service の再起動や API 実リクエストは未実施。
- `ui-dist/` は 67 ファイル、約 8.8MB。
- `docs/images/llama-ui-settings-ja.png` は 1242x1181 の PNG。
- OpenAI 互換 API の JSON にある `"model"` は `/v1/models` で見えるモデル ID を指定するものとして記載した。
