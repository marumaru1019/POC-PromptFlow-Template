

# Query Expansion Flow with PromptFlow

## 概要

このプロジェクトは、Azure OpenAIを活用してデータ処理フローを構築するためのガイドラインを提供します。AI StudioとPromptFlowを組み合わせることで、効率的にデータ検索、処理、そして結果の生成を行います。

## 事前準備

- GitHubアカウント
- Azureサブスクリプション
- RAG に使用したいデータ (Option)

## クイックスタート

### 1. リポジトリのフォーク

1. GitHubでこのプロジェクトのリポジトリを開きます。
2. 右上の「Fork」ボタンをクリックし、リポジトリを自分のアカウントにフォークします。

### 2. CodeSpacesのセットアップ

1. フォークしたリポジトリにアクセスします。
2. 「Code」ボタンをクリックし、「Create codespace on main」を選択して新しいCodeSpaceを作成します。
![image](https://github.com/user-attachments/assets/a2572eed-0130-4f16-b670-31ad01646b1a)


### 3. Python仮想環境の作成と依存関係のインストール

CodeSpaceが立ち上がったら、以下のコマンドを実行してPython仮想環境を作成し、依存関係をインストールします。

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

また、以下のコマンドを使用してAzurf CLIをインストールします。
また、ml拡張機能をインストールします。

```:bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
az extension add -n ml
```

### 4. Azureにログイン

Azure CLIを使用して、Azureアカウントにログインします。

```bash
az login --use-device-code
```

上記のコマンドを実行すると、下記のようなログが表示されます。[ブラウザ](https://microsoft.com/devicelogin)にアクセスして、表示されるコードを入力してください。

```bash
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code <コード> to authenticate.
```

### 5. 必要なAzureリソースの作成
scripts/project.sh の中を確認すると、環境変数をセットする箇所があるので任意の値に変更してください(設定する環境変数は、`RESOURCE_GROUP_NAME`、`HUB_NAME`、`PROJECT_NAME`、`OPENAI_SERVICE_NAME`、`SEARCH_SERVICE_NAME`です)。


```:scripts/project.sh
～～～

SUBSCRIPTION_ID=$(az account show --query id -o tsv)
RESOURCE_GROUP_NAME=<任意のリソースグループ名>
LOCATION="eastus"
HUB_NAME=<任意のHub名>
PROJECT_NAME=<任意のプロジェクト名>
OPENAI_SERVICE_NAME=<任意のOpenAIのサービス名>
SEARCH_SERVICE_NAME=<任意のAI Searchのサービス名>

～～～
```


環境変数をセットしたら、以下のコマンドを実行してAzureリソースを作成します。

```bash
sh scripts/create_project.sh
```

デプロイが完了すると、下記のリソースが作成されます。
Azure Portal上からすべてのリソースが作成されていることを確認してください。

- Azure AI Hub
- Azure AI Project
- Azure AI Services
- Azure Key Vault
- Storage Account
- Azure AI Search

![image](https://github.com/user-attachments/assets/90b672bb-9390-47f5-a7e1-0e8105cef1df)

また、ルートディレクトリに`.env`が作成されます。このファイルには、Azureリソースの情報が記載されています。


### 6. Azure AI Studioで接続の作成
下記の手順でAzure AI StudioでOpenAIとAI Searchの接続を作成します。
1. [Azure AI Studio](https://ai.azure.com/) にアクセスします。
2. アクセスできたら、すべてのプロジェクトを表示をクリックして、5で作成したプロジェクト名を選択します。
![image](https://github.com/user-attachments/assets/3669bf1c-529c-4aed-b3a7-07d414b7ca46)
![image](https://github.com/user-attachments/assets/d924c9a4-c5a9-421a-9b6f-df1e479fd1a5)

3. 左のナビゲーションバーから「設定」をクリックし、新しい接続を作成します。
![image](https://github.com/user-attachments/assets/94bd9d8b-ffe6-4923-8d69-071455af0dd6)

4. OpenAIの接続を作成します。接続一覧から、「Azure AI Services」を選択し、5で作成したリソースに対して「接続を追加する」をクリックします。
![image](https://github.com/user-attachments/assets/c7122d0e-489f-44d8-9d27-40c2c179e4b6)
![image](https://github.com/user-attachments/assets/56d850db-eb33-45f5-96e8-8b13486a802a)

5. 同様の手順でAI Searchの接続を作成します。接続一覧から、「Azure AI Services」を選択し、5で作成したリソースに対して「接続を追加する」をクリックします。
![image](https://github.com/user-attachments/assets/4f78fc51-879b-4b50-8c42-e278c245ef7e)


### 7. 環境変数の設定

プロジェクトのルートディレクトリにある`.env`ファイルに、以下の環境変数を追加します。

```env
AZURE_OPENAI_CONNECTION_NAME=your_openai_connection_name
AZURE_SEARCH_CONNECTION_NAME=your_search_connection_name
```

`your_openai_connection_name`と`your_search_connection_name`は、Azure AI Studioで作成した接続の名前に置き換えてください。
接続名は、AI Studioでプロジェクトを選択した後に、左のナビゲーションバーから「設定」をクリックし、接続の一覧を表示し、該当の接続を選択することで確認できます。
![image](https://github.com/user-attachments/assets/1fc74d71-5672-4cfe-94a8-0954e05319ed)
![image](https://github.com/user-attachments/assets/398bc09f-db5a-45ef-82e6-9e465bfdbf5c)

### 8. インデックスの作成

データに基づいてインデックスを作成します。
indexing/dataディレクトリにインデックス化したいデータを配置してください。
もし、配置したいデータがない場合、ダミーデータとして就業規則のデータが用意されているのでそのままでも構いません。
以下のコマンドを実行することで、インデックスを作成します。


```bash
python -m indexing.build_index --index-name <任意のインデックス名> --path-to-data indexing/data
```

インデックス名を環境変数に追加します。

```env
AZUREAI_SEARCH_INDEX_NAME=your_index_name
```

### 9. Flowの作成とZIP化

src/flow_templateにクエリ拡張のフローが用意されているので、それをアップロードする準備を行います。
以下のコマンドを実行して、フローに今回作成した環境の内容を反映させ、ZIP化します。

```bash
python scripts/create_flow.py
python scripts/create_flow_zip.py
```

これにより、`flow_template.zip`がプロジェクトのルートディレクトリに生成されます。

### 10. Flowのアップロード

1. AI Studioにアクセスします。
2. 左のナビゲーションバーから「プロンプトフロー」を選択し、作成をクリックしてください。下にスクロールするとインポートのセクションがあるので、そこにある「ローカルからアップロード」を選択し、ファイルのアップロードを「Zipファイル」を選択して、9で作成したZIPファイルをアップロードしてください。
フォルダー名は任意の名前を設定し、フローの種類は「Standard Flow」に設定してください。
![image](https://github.com/user-attachments/assets/0848a2d3-f297-4d2b-9240-b1443c3e6ee6)
![image](https://github.com/user-attachments/assets/62a172c4-beb8-4c38-8cdb-1a3129f0e9e0)

フローをインポートすると、以下のような画面が表示されます。
![image](https://github.com/user-attachments/assets/1b747ceb-b614-4a5c-9df1-2e0752dc0b66)


### 11. 動作確認

1. AI Studioでコンピューティングセッションをオンにします。
![image](https://github.com/user-attachments/assets/a0dc5800-44f5-4362-8d09-8749374c0b4c)

2. フローを実行し、トレース結果を確認します。
![image](https://github.com/user-attachments/assets/f0da4a0d-9a54-4d12-9b79-fc76a13e2e3a)
![image](https://github.com/user-attachments/assets/3b7bb274-e19d-405f-afba-aa644ad2c95e)

## ローカル環境での構築 (オプション)

ローカルでの動作確認も可能ですが、一部の機能に制約があります。

### 接続の作成

以下のコマンドを使用して、ローカル環境で接続を作成します。

```bash
pf connection create -f connection/openai.yml
pf connection create -f connection/search.yml
```

### 制約事項

ローカル環境では、`lookup`機能が正常に動作しない可能性があります。詳細は[GitHub issue #3632](https://github.com/microsoft/promptflow/issues/3632)をご確認ください。

## その他の考慮事項

- **開発環境の依存関係管理**: ライブラリのバージョンを固定することを推奨します。
- **セキュリティ対策**: 環境変数やAPIキーなどの機密情報は、適切に管理してください。
- **CI/CDの設定**: 自動デプロイの設定を行うと、作業効率が向上します。

---

この手順書を参考に、プロジェクトのセットアップをスムーズに進めてください。