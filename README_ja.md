# stable-diffusion-webui-container

VSCode Dev Containers を使用して Stable Diffusion WebUI を Docker コンテナ内で実行するための環境です。

## 特徴

* 最近の NVIDIA GPU (Blackwell 世代)対応
* SDP Attention を使用 (xformers は未使用)
* VSCode Dev Containers ワークフロー - Dockerfile のカスタマイズ不要
* モデルと出力の bind-mount
* 最小限のセットアップ - devcontainer.json のみの設定で動作

これは初心者向けのワンクリックインストーラーではありません。少なくとも Docker、VSCode Dev Containers、Linux コマンドラインの基本的な知識が必要です。

---

## 要件

このリポジトリは以下に関する基本的な知識を前提としています：

* Docker
* VSCode Dev Containers
* Linux コマンドライン操作

### ホスト（GPU 搭載 PC）の必須要件

* NVIDIA GPU
* NVIDIA ドライバ（最新）
* NVIDIA Container Toolkit
* Docker

### ローカル PC（VSCode 実行環境）の必須要件

* VSCode
* Dev Containers 拡張機能

**リモート接続の場合:** ホストとローカル PC が異なるマシンの場合、ローカル PC で VSCode と Dev Containers 拡張機能を実行し、ホストで Docker と NVIDIA Container Toolkit が動作していれば利用可能です。

---

## テスト環境

このコンテナは以下のホストで動作確認済みです：

| コンポーネント | バージョン |
| --- | --- |
| ホスト OS | Ubuntu Server 26.04 LTS |
| GPU | NVIDIA GeForce RTX 5080 16GB |
| NVIDIA ドライバ | 595.71.05 |
| Docker | 29.5.0 build 98f1464 |
| NVIDIA Container Toolkit | インストール済み |
| VSCode | Latest |
| Dev Containers 拡張機能 | Latest |

---

## GPU / CUDA に関する注意

このコンテナはホストにインストールされた NVIDIA ドライバに依存しています。

以下を確認してください：

* NVIDIA ドライバが正しくインストールされているか
* NVIDIA Container Toolkit が設定されているか
* Docker から GPU にアクセスできるか

以下を使用して確認できます：

```bash
docker run --rm --gpus all debian:stable-slim nvidia-smi
```

GPU アクセスが正しく機能していれば、コンテナ内の `nvidia-smi` は GPU 情報を表示します。

## クイックスタート

### 1. リポジトリをクローン

```bash
git clone https://github.com/ain1084/stable-diffusion-webui-container-launcher
cd stable-diffusion-webui-container-launcher
```

### 2 devcontainer.json の作成

#### 2.1 devcontainer.json をコピー

`.devcontainer` ディレクトリに [`devcontainer.example.json`](./.devcontainer/devcontainer.example.json) が用意されています。これを `.devcontainer/devcontainer.json` にコピーしてください。

**重要：** `devcontainer.json` の作成は必須です。この設定ファイルがないと、VSCode が Dev Container を起動できません。

#### 2.2 マウント設定の確認・カスタマイズ（オプション）

`devcontainer.json` にはデフォルトで以下のマウント設定が含まれています。これらは、ホスト上での保持を推奨するディレクトリの一例です：

```text
Repository Root
└── stable-diffusion-webui/
    ├── models/Stable-diffusion/   # Model checkpoints
    ├── models/Lora/               # LoRA models
    ├── extensions/                # Extensions directory (independent of WebUI core)
    ├── outputs/                   # Generated images
    └── log/                       # Logs
```

**注記：** このリポジトリをクローンすると、これらのディレクトリはすでに存在しています（`.gitignore` ファイル付き）。追加で `mkdir -p` を実行する必要はありません。

マウント設定により、以下のようにホスト側のワークスペースからコンテナ内に bind-mount されます：

| ホスト（ワークスペース） | コンテナ内のパス | 目的 |
| --- | --- | --- |
| `stable-diffusion-webui/models/Stable-diffusion` | `/opt/stable-diffusion-webui/models/Stable-diffusion` | モデルチェックポイント |
| `stable-diffusion-webui/models/Lora` | `/opt/stable-diffusion-webui/models/Lora` | LoRA モデル |
| `stable-diffusion-webui/extensions` | `/opt/stable-diffusion-webui/extensions` | 拡張機能用ディレクトリ |
| `stable-diffusion-webui/outputs` | `/opt/stable-diffusion-webui/outputs` | 生成画像 |
| `stable-diffusion-webui/log` | `/opt/stable-diffusion-webui/log` | ログファイル |

**マウント設定のカスタマイズ：**

モデルがホストの別の場所に保存されている場合や、他のディレクトリをマウントしたい場合は、`devcontainer.json` のマウント設定を編集してください。

**例：** モデルがホストの `~/shared/sd/models/` にある場合:

```json
"mounts": [
    "source=${localEnv:HOME}/shared/sd/models,target=/opt/stable-diffusion-webui/models/Stable-diffusion,type=bind",
    // ... etc
]
```

※ `${localEnv:HOME}` はホスト上のユーザーのホームディレクトリを指します。

なお、マウント設定を変更しない場合でも起動は可能ですが、初回起動時はモデルディレクトリ(`stable-diffusion-webui/models/Stable-diffusion`) が空の状態となっているため、WebUI によってデフォルトモデル（約3GB）がダウンロードされます。

**マウント設定変更後の再起動：**

`devcontainer.json` のマウント設定を変更した場合は、設定を反映するために Dev Container の再生成が必要です。

変更後は VSCode のコマンドパレット（Ctrl+Shift+P）から以下を実行してください：

- `Dev Containers: Rebuild Container`

これにより、新しいマウント設定でコンテナが再作成されます。

※ `Reopen in Container` では変更が反映されない場合があります。

**重要な注意：**

* **マウント設定は後から追加・変更できます**  
  必要に応じて `devcontainer.json` を編集してください。変更後は `Rebuild Container` が必要です。

* **コンテナ内だけに保存されたファイルは失われる場合があります**  
  `Rebuild Container` を実行すると、新しい設定でコンテナが再作成されます。bind mount していないディレクトリ内のデータは失われる可能性があります。

* **重要なデータはホスト(ワークスペース)側へ保存してください**  
  モデル、生成画像、設定ファイルなど、保持したいデータは bind mount を使用してホスト側ディレクトリへ保存することを推奨します。

#### 2.3 PyTorch インストールコマンドのカスタマイズ（オプション）

デフォルトでは、`pip install torch` により最新版の PyTorch がインストールされます。

古い GPU では、最新版の PyTorch が動作しない場合があります。その場合は、対応する CUDA バージョンを指定してください。

**設定方法：**

`devcontainer.json` の `args` セクション内で `TORCH_INSTALL_COMMAND` を指定します：

```json
"args": {
    "TORCH_INSTALL_COMMAND": "pip install torch --extra-index-url https://download.pytorch.org/whl/cu124"
}
```

**GPU 世代別の推奨 CUDA バージョン：**

以下の表を参考にして目安にしてください。

| GPU 世代 | 推奨 CUDA バージョン | 設定例 |
| --- | --- | --- |
| Pascal 世代 (GeForce GTX 10xx 系など) | cu121 / cu124 | `pip install torch --extra-index-url https://download.pytorch.org/whl/cu124` |
| RTX 20/30/40/50 系 | 現行最新版 | `pip install torch`（デフォルト） |

### 3. VSCode で開き、コンテナで再度開く

1. VSCode でリポジトリフォルダを開きます
2. プロンプトが表示されたら、**「Reopen in Container」** をクリックするか、**Ctrl+Shift+P** を使用して「Reopen in Container」と入力します
3. コンテナが自動的にビルドおよび起動します（初回実行時は数分かかる場合があります）

### 4. WebUI を実行

コンテナの準備が完了したら、ターミナルに表示される内容 (WELCOME.txt) に従って、WebUI を起動してください。

以下の操作で WebUI を起動するタスクを実行できます：

```text
Ctrl+Shift+B
```

### 5. WebUI にアクセス

WebUI が起動すると、ブラウザから以下の URL でアクセスできます：

```text
http://localhost:7860
```

この環境では VSCode Dev Containers のポート転送機能を利用しています。ローカル側でポート 7860 が既に使用されている場合は、自動的に別のポート番号へ転送されることがあります。

実際に使用されている URL は、VSCode の「PORTS」タブから確認できます。

**再起動方法:** ターミナルで Ctrl+C を押して WebUI を停止してから、Ctrl+Shift+B で再度タスクを実行してください。

**ブラウザで開けない場合:** VSCode の「PORTS」タブを確認し、ポート 7860 が転送されていることを確認してください。自動ポートフォワーディングが機能していない場合は、手動でポート転送を追加する必要があります。

## カスタマイズ

### 1. stable-diffusion-webui コミットの変更

stable-diffusion-webui のコミットは `SD_WEBUI_COMMIT` ビルド引数を使用してピン留めされています：

```json
"args": {
    "SD_WEBUI_COMMIT": "1937682a20f7f0442311a1ede68f9f0cb480163b"
}
```

[.devcontainer/devcontainer.json](.devcontainer/devcontainer.json) でこれを修正して、別のコミットを使用できます。

**重要：** `SD_WEBUI_COMMIT` を変更した場合、**コンテナの再ビルド（Rebuild Container）** が必要です。単に WebUI の再起動では変更は反映されません。VSCode で「Rebuild Container」コマンドを実行してください。

### 2. 起動オプション / タスク

この環境では WebUI の起動オプションは VSCode タスクで管理しています。webui-user.sh を変更するのではなく、VSCode タスク([`.vscode/tasks.json`](.vscode/tasks.json))を編集して起動オプションを追加または変更してください。

以下の起動オプションを指定しています。

* `--opt-sdp-attention` : xformers を使用せず、PyTorch のネイティブな SDP Attention を有効にします([xformers について](#3-xformers-について) のセクションも参照)。
* `--api`: WebUI の REST API を有効にします。外部アプリケーションから WebUI の API を利用できます。

`--listen` オプションは使用していません。VSCode Dev Containers のポート転送機能を利用して、VSCode を実行しているローカルマシンから WebUI にアクセス可能なためです。

### 3. xformers について

この環境は xformers をインストール、使用しません。

最近の NVIDIA GPU 環境では、xformers は互換性問題を起こす場合があるためです。特に Blackwell 世代では動作しない、build が必須、CUDA mismatch といった問題が生じることがあります。

代わりに、この環境は PyTorch SDP attention オプションを設定しています([起動オプション](#2-起動オプション--タスク)のセクションを参照)。

### 4. 拡張機能

拡張機能はプリインストールされていません。`extensions` ディレクトリはデフォルトで bind-mount されているため、WebUI 上からもインストール用に利用可能です。また、目的の拡張機能を `stable-diffusion-webui/extensions/` へ配置することで、手動で拡張機能を追加できます。詳しくはそれぞれの拡張機能のドキュメントを参照してください。

---

## ライセンス

このリポジトリの設定ファイルおよび Docker 関連ファイルは MIT License の下で公開されています。

このリポジトリの主な依存ソフトウェアについては、以下のライセンスに従ってください：

* [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) - [LICENSE](https://github.com/AUTOMATIC1111/stable-diffusion-webui/blob/master/LICENSE.txt)
