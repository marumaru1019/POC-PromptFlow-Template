# 構築手順
リポジトリをフォーク
CodeSpacesから新しいCodeSpaceを作成する

# Pythonの仮想環境の作成とライブラリのインストール
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt

# azure にログイン
az login --use-device-code

# 必要なリソースの作成
sh scripts/project.sh

# 接続の作成
UI上の操作

## 環境変数ファイルの追記
AZURE_OPENAI_CONNECTION_NAM
AZURE_SEARCH_CONNECTION_NAME

# インデックスの作成
python -m indexing.build_index --index-name <任意のインデックス名> --path-to-data=indexing/data

## 環境変数ファイルの追記
AZUREAI_SEARCH_INDEX_NAME

# flowの作成&zip化
python scripts/create_flow.py
python scripts/create_flow_zip.py 

ルートディレクトリにflow_template.zipが出来上がる。

# flowのアップロード
AI Studioに移動し、プロンプトフローからローカルアップロードを行う。

# 動作確認
1. コンピューティングセッションON
2. 実行
3. traceの確認

# ローカル構築 (Option)
ローカルでlookupが使えなそうなので、現地点では使用不可能
pf connection create -f connection/openai.yml
pf connection create -f connection/search.yml

ローカルでlookupが使えなそう
https://github.com/microsoft/promptflow/issues/3632