# 引き継ぎ情報

## 現状

- README に llama.cpp の設定手順をまとめ済み。
- GGUF 内蔵の chat template で足りるモデルは `chat-template-file` を書かない。今は 6 つのプリセットが `models/` のテンプレートを指定している（一覧は README の「chat-template の扱い」）。
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

- GPU（CUDA）での動作は確認済み（2026-11-16、ビルド版 v0.9.0）。
- Hugging Face からのモデル取得は初回に時間とディスク容量を使う。
- `chat-template-file` はカスタムテンプレートが必要なプリセットだけで使い、テンプレートは `models/` に置く。
- `llama.cpp.service` の `--path` は実運用では `/home/onoue/src/oss/llama.cpp/tools/ui/dist` を参照している。`ui-dist/` は参考用コピー。
- `--path /home/onoue/src/oss/llama.cpp/tools/ui/dist` は日本語化済み Web UI dist を使うための指定。
- Offline-llm-translator ブラウザ拡張機能のオリジンは、PC ごとの `~/.config/llama.cpp/llama.env` の `LLAMA_ARG_CORS_ORIGINS` で許可する。
- `--ui-mcp-proxy` は llama-server の Web UI から MCP を使うための指定。
- 現在の `--tools` は `read_file,file_glob_search,grep_search` のみ（v0.9.0 に `get_datetime` はない）。書き込み系の `exec_shell_command`、`write_file`、`edit_file` は有効化していない。
- `write_file` / `edit_file` は `llama-server` の実行ユーザー権限でファイルを書き込む。systemd user service では通常 `onoue` 権限になり、llama.cpp 側に専用 sandbox や書き込み先制限はない。
- `mcp-searxng` は HTTP mode で起動し、Web UI には `http://127.0.0.1:3000/mcp` を `useProxy = true` で登録する想定。
- SearXNG 側は JSON format が有効である必要がある。
- Memos MCP は Memos 本体の `https://memos.showway.biz/mcp` を使う。Web UI 側では Authorization を有効にして Bearer PAT を設定する。
- Memos MCP も `useProxy = true` で llama-server proxy 経由にする想定。
- 次に試す MCP 候補は fetch / URL reader、filesystem read-only、git read-only の順。stdio 専用 server は Web UI に直接 URL 登録できないため、HTTP proxy / adapter が必要。
- MCP server を増やす場合は read-only、ローカル待ち受け、許可ディレクトリ限定、token は環境変数渡しを基本にする。
