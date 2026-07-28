# llama.cpp 設定手順

このメモは `/etc/llama.cpp/models.ini` と
`/home/onoue/.config/systemd/user/llama.cpp.service` を使って、
`llama-server` を systemd user service として動かすための手順。

## 前提

- `llama-server` が `/usr/bin/llama-server` にある
- llama.cpp は AUR パッケージ `llama.cpp-cuda` で入れる
- モデル設定ファイルを `/etc/llama.cpp/models.ini` に置く
- Jinja チャットテンプレートを `/home/onoue/.local/lib/models/` に置く
- llama.cpp の Web UI が `/home/onoue/src/oss/llama.cpp/tools/ui/dist` にある
- server は `127.0.0.1:8080` で待ち受ける

## インストール

Arch Linux では AUR の `llama.cpp-cuda` を使う。

```sh
yay -S llama.cpp-cuda
```

インストールされた場所を確認する。

```sh
command -v llama-server
pacman -Ql llama.cpp-cuda | grep '/llama-server$'
```

この設定では `/usr/bin/llama-server` を systemd から起動する。

## GPU/CUDA 確認

まず NVIDIA ドライバが見えているか確認する。

```sh
nvidia-smi
```

GPU 名、Driver Version、CUDA Version、VRAM 使用量が出ればドライバ側は見えている。
`NVIDIA-SMI has failed` が出る場合は、llama.cpp 以前に NVIDIA ドライバかカーネルモジュール側の問題。

`llama-server` のビルド情報を確認する。

```sh
llama-server --version
```

CUDA 対応ビルドかどうかは、起動ログで確認するのが確実。

```sh
llama-server -hf unsloth/gemma-4-E4B-it-GGUF:Q4_K_M \
  --n-gpu-layers 1 \
  --ctx-size 512 \
  --no-webui
```

CUDA が使えていれば、起動ログに CUDA / GPU backend の初期化ログが出る。
`failed to initialize CUDA` や `compiled without GPU support` が出る場合は、CUDA が使えていない。

## ディレクトリを作る

```sh
sudo mkdir -p /etc/llama.cpp
mkdir -p /home/onoue/.config/systemd/user
mkdir -p /home/onoue/.local/lib/models
```

## chat-template を配置する

このリポジトリの `models/` に、`models.ini` から参照する Jinja テンプレートを置いている。

```text
models/
├── Ornith-1.0-9B-GGUF.jinja
├── gemma-4-E4B-it-GGUF.jinja
├── gemma-4.jinja
└── gemma.jinja
```

設定先へコピーする。

```sh
cp models/*.jinja /home/onoue/.local/lib/models/
```

`models.ini` の `chat-template-file` は `/home/onoue/.local/lib/models/*.jinja` を参照する。
テンプレートを更新したら、`llama.cpp.service` を再起動する。

## モデル設定

`/etc/llama.cpp/models.ini` を作成する。

```ini
version = 1

[gemma-4-E4B-it]
hf = unsloth/gemma-4-E4B-it-GGUF:Q4_K_M
chat-template-file = /home/onoue/.local/lib/models/gemma-4-E4B-it-GGUF.jinja
n-gpu-layers = all
ctx-size = 393216
parallel = 4
stop-timeout = 10
spec-type = draft-mtp
spec-draft-n-max = 2
temp = 1.0
top-p = 0.95
top-k = 64

[ornith-1.0-9b]
hf-repo = deepreinforce-ai/Ornith-1.0-9B-GGUF
hf-file = ornith-1.0-9b-Q4_K_M.gguf
chat-template-file = /home/onoue/.local/lib/models/Ornith-1.0-9B-GGUF.jinja
n-gpu-layers = all
ctx-size = 65536
parallel = 1
stop-timeout = 10

[bonsai-27b]
hf-repo = prism-ml/Bonsai-27B-gguf
hf-file = Bonsai-27B-Q1_0.gguf
n-gpu-layers = all
ctx-size = 65536
parallel = 1
stop-timeout = 10

[translategemma-4b]
hf = mradermacher/translategemma-4b-it-GGUF:Q4_K_M
chat-template-file = /home/onoue/.local/lib/models/gemma.jinja
ctx-size = 16384
parallel = 4
n-gpu-layers = all
stop-timeout = 10

[translategemma-12b]
hf = mradermacher/translategemma-12b-it-GGUF:Q4_K_M
chat-template-file = /home/onoue/.local/lib/models/gemma.jinja
ctx-size = 16384
parallel = 4
n-gpu-layers = all
stop-timeout = 10

[translategemma-4b-translate]
hf = mradermacher/translategemma-4b-it-GGUF:Q4_K_M
chat-template-file = /home/onoue/.local/lib/models/gemma.jinja
ctx-size = 4096
parallel = 1
n-gpu-layers = 99
stop-timeout = 10

[translategemma-12b-translate]
hf = mradermacher/translategemma-12b-it-GGUF:Q4_K_M
chat-template-file = /home/onoue/.local/lib/models/gemma.jinja
ctx-size = 4096
parallel = 1
n-gpu-layers = 99
stop-timeout = 10
```

### モデル指定の使い分け

`hf = repo:file-or-quant` は Hugging Face から取得する指定。

```ini
hf = unsloth/gemma-4-E4B-it-GGUF:Q4_K_M
```

`hf-repo` と `hf-file` はリポジトリと GGUF ファイル名を分ける指定。

```ini
hf-repo = deepreinforce-ai/Ornith-1.0-9B-GGUF
hf-file = ornith-1.0-9b-Q4_K_M.gguf
```

## systemd user service

`/home/onoue/.config/systemd/user/llama.cpp.service` を作成する。

```ini
[Unit]
Description=llama.cpp server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/llama-server \
  --host 127.0.0.1 \
  --port 8080 \
  --models-preset /etc/llama.cpp/models.ini \
  --models-max 1 \
  --jinja \
  --path /home/onoue/src/oss/llama.cpp/tools/ui/dist \
  --tools read_file,file_glob_search,grep_search,get_datetime
Restart=on-failure
RestartSec=3

[Install]
WantedBy=default.target
```

`--models-max 1` は同時ロードするモデル数の上限。VRAM を抑えたい場合はこのままでよい。

`--jinja` は `chat-template-file` を使うために必要。

`--path /home/onoue/src/oss/llama.cpp/tools/ui/dist` は llama.cpp の Web UI を差し替えるために使う。
この設定では、日本語化済みの Web UI dist を指定している。

`--tools` は llama.cpp のツール呼び出しで使えるツールを指定する。

日本語化された Web UI の設定画面例。

![llama.cpp Web UI 日本語設定画面](docs/images/llama-ui-settings-ja.png)

この service を `systemctl --user` で常駐させておくと、`models.ini` に書いたモデル定義を
router server が読む。Hugging Face のモデルは、各セクションで `hf = repo:quant` の形で指定する。

```ini
[gemma-4-E4B-it]
hf = unsloth/gemma-4-E4B-it-GGUF:Q4_K_M
```

API リクエストで指定するモデル名は、起動後に `/v1/models` で確認する。

```sh
curl http://127.0.0.1:8080/v1/models
```

手元で毎回 `llama serve -hf repo/model:quant` を起動し直さなくて済むので、
Local Translator、xTranslator、OpenAI 互換クライアントから使いやすい。

## サービスを反映する

```sh
systemctl --user daemon-reload
systemctl --user enable --now llama.cpp.service
```

ログアウト後も user service を動かしたい場合は linger を有効にする。

```sh
sudo loginctl enable-linger onoue
```

SSH セッションやデスクトップセッションを閉じても `llama.cpp.service` を常駐させたいなら、ほぼ必要。

設定を変えたあとに再起動する。

```sh
systemctl --user restart llama.cpp.service
```

## 状態確認

```sh
systemctl --user status llama.cpp.service
journalctl --user -u llama.cpp.service -f
```

API の疎通確認。

```sh
curl http://127.0.0.1:8080/v1/models
```

OpenAI 互換 API の実行確認。

```sh
curl http://127.0.0.1:8080/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "translategemma-4b-translate",
    "messages": [
      {
        "role": "user",
        "content": "こんにちは。短く自己紹介して。"
      }
    ],
    "temperature": 0.7,
    "max_tokens": 128
  }'
```

JSON の `"model"` には `/v1/models` で見えるモデル ID を指定する。

Web UI はブラウザで開く。

```text
http://127.0.0.1:8080
```

## モデル保存先

`llama serve -hf モデル名` を使うと、Hugging Face からモデルがダウンロードされる。

```sh
llama serve -hf unsloth/gemma-4-E4B-it-GGUF:Q4_K_M
```

この場合、モデルは Hugging Face のキャッシュ配下に保存される。

```text
/home/onoue/.cache/huggingface/
```

Hugging Face 上のモデルが更新された場合も、キャッシュ側は必要に応じて更新される。
モデル取得と更新確認を llama.cpp 側に任せられるので、手で GGUF を探して落としてくる手間が減る。

この README の systemd 設定では、Hugging Face から取得するモデルは
`hf` / `hf-repo` / `hf-file` に任せる。
`/etc/llama.cpp/models.ini` には、各セクションごとにモデル取得元を書く。

```text
[translategemma-4b-translate]
hf = mradermacher/translategemma-4b-it-GGUF:Q4_K_M
```

キャッシュ内のモデル一覧はこれで確認する。

```sh
llama-server --cache-list
```

ディスク使用量を見る。

```sh
du -sh /home/onoue/.cache/huggingface
```

GGUF は数 GB から数十 GB になる。複数モデルを試すなら、空き容量を先に見ておく。

## VRAM 調整指針

VRAM 使用量に効く主な設定は `ctx-size`、`parallel`、`n-gpu-layers`、`--models-max`。

- `ctx-size` はコンテキスト長。大きいほど KV cache が増える。
- `parallel` は同時処理数。増やすとスロット分のメモリが増える。
- `n-gpu-layers` は GPU に載せるレイヤ数。`all` は可能な限り GPU に載せる。
- `--models-max` は同時ロードするモデル数。この設定では `1` にして VRAM を抑えている。

起動に失敗する、または推論中に VRAM が足りない場合は、まずこの順で下げる。

1. `parallel` を `1` にする
2. `ctx-size` を下げる
3. `n-gpu-layers = all` を数値指定にする
4. 同時に使わないモデルを unload する、または `--models-max 1` を維持する
5. 量子化の軽い GGUF に変える

翻訳用の `translategemma-4b-translate` / `translategemma-12b-translate` は、
`ctx-size = 4096`、`parallel = 1` にして VRAM を抑える設定。

## 翻訳用モデルの考え方

`translategemma-4b-translate` と `translategemma-12b-translate` は
Local Translator や xTranslator 向けの軽量設定。

- `ctx-size = 4096` で KV cache の使用量を抑える
- `parallel = 1` で同時処理数を抑える
- `n-gpu-layers = 99` で GPU 使用量を固定寄りにする
- 短文は 4B、長文やエラー時は 12B という使い分けを想定する

通常チャット用の `translategemma-4b` と `translategemma-12b` は
`ctx-size = 16384`、`parallel = 4` にしているので、翻訳専用設定よりメモリを使う。

## 注意点

- `ctx-size` と `parallel` を大きくすると KV cache が増えて VRAM 使用量も増える。
- `n-gpu-layers = all` は可能な限り GPU に載せる設定。VRAM が足りない場合は数値指定に落とす。
- `hf` / `hf-repo` / `hf-file` を使うモデルは、初回起動時にダウンロードが必要になる。
