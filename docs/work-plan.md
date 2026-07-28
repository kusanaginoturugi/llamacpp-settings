# 作業計画

## 目的

llama.cpp を systemd user service で運用するための設定手順を README にまとめる。

## 対応内容

- `/etc/llama.cpp/models.ini` の設定例を文書化する
- `/home/onoue/.config/systemd/user/llama.cpp.service` の設定例を文書化する
- chat-template ファイルをリポジトリに同梱する
- AUR `llama.cpp-cuda` での導入手順を追記する
- GPU/CUDA の確認手順を追記する
- Hugging Face キャッシュとモデル更新の説明を追記する
- VRAM 調整指針を追記する
- `systemctl --user` で常駐させる利点を追記する
- OpenAI 互換 API の確認例を追記する

## 完了条件

- README だけで llama.cpp service の設定・起動・疎通確認まで追える
- chat-template ファイルを別途探さずに配置できる
- commit/push 前に作業記録と引き継ぎ情報が残っている
