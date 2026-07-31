# 引き継ぎ情報

## 現状

- README に llama.cpp の設定手順をまとめ済み。
- 通常運用では `models.ini` に `chat-template-file` を書かず、GGUF 内蔵の chat template を使う方針。
- llama.cpp Web UI の dist は参考用として `ui-dist/` に同梱済み。
- README に日本語化済み Web UI の設定画面スクリーンショットを掲載済み。
- systemd user service の実ファイルは `/home/onoue/.config/systemd/user/llama.cpp.service` にあり、`--path` 行の継続 `\` は修正済み。

## 反映が必要な場合

README や service を変更したあと、実運用へ反映するには次を実行する。

```sh
systemctl --user daemon-reload
systemctl --user restart llama.cpp.service
```

ログアウト後も常駐させるなら次を使う。

```sh
sudo loginctl enable-linger onoue
```

## 注意

- この環境では CUDA が見えていないため、GPU 動作確認は未完了。
- Hugging Face からのモデル取得は初回に時間とディスク容量を使う。
- `chat-template-file` はカスタムテンプレートが必要な場合だけ使う。通常は書かない。
- `llama.cpp.service` の `--path` は実運用では `/home/onoue/src/oss/llama.cpp/tools/ui/dist` を参照している。`ui-dist/` は参考用コピー。
- `--path /home/onoue/src/oss/llama.cpp/tools/ui/dist` は日本語化済み Web UI dist を使うための指定。
