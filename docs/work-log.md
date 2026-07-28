# 作業記録

## 2026-07-28

- `/etc/llama.cpp/models.ini` と `llama.cpp.service` の実設定を確認した。
- `README.md` を作成し、モデル設定、systemd user service、反映手順、API 疎通確認を記載した。
- `/home/onoue/.config/systemd/user/llama.cpp.service` の `--path` 行に継続用の `\` を追加した。
- `/home/onoue/.local/lib/models/` から chat-template 4 件を `models/` にコピーした。
- `models/` のテンプレートとコピー元の SHA256 が一致することを確認した。
- README に AUR `llama.cpp-cuda`、GPU/CUDA 確認、linger、OpenAI 互換 API、Hugging Face キャッシュ、VRAM 調整指針を追記した。
- README から不要な `ExecStart` 継続行の注意書きを削除した。

## 確認事項

- `llama serve --help` で `-hf`、`HF_TOKEN`、`--no-webui`、`--cache-list` が使えることを確認した。
- この作業環境では `nvidia-smi` が NVIDIA driver に接続できず、`llama-server --version` でも CUDA 初期化エラーが出た。
- service の再起動や API 実リクエストは未実施。
