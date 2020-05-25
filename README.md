# GCP IoT Core ハンズオン
IoT CoreからPub/Sub、Dataflowを経由して、最終的にBigQueryデータを格納するところまで

## 実行環境
以下をインストールしておいてください。
- python3実行環境
- virtualenv
- pip

## STEP.1 デバイスからPub/Subへ通信
### デバイスレジストリの作成 @IoT Core
1. [コンソール](https://console.cloud.google.com/iot)へ移動
2. 上部の**レジストリの作成**をクリック
3. **レジストリID**に任意のIDを入力(ex. yourname-registry)
4. 近いリージョンを選択(ex. us-central1) 
5. **Cloud Pub/Sub トピック**はプルダウンから*トピックを作成する*を選択
6. **トピックID**に任意のIDを入力(ex. yourname-device-events)
7. 他はデフォルトのまま、**トピックの作成**をクリック
8. **詳細設定の表示**をクリック
9. **プロトコル**は*MQTT*を選択
10. 他はデフォルトのまま、**作成**をクリック

### 証明書の作成 @ローカル環境 or Cloud Shell 
以下のコマンドを実行
```
openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes -out rsa_cert.pem -subj "/CN=unused"
```

### デバイスの登録 @IoT Core
1. [コンソール](https://console.cloud.google.com/iot)へ移動
2. 左のメニューから**デバイス**をクリック
3. 上部の**デバイスを作成**をクリック
4. **デバイスID**に任意のIDを入力(ex. yourname-device)
5. **通信、STACKDRIVER LOGGING、認証**をクリック
6. 認証（オプション）の**公開鍵の形式**で*RS256_X509*を選択
7. 先程作成した**rsa_cert.pem**の内容をコピペする
8. 他はデフォルトのまま、**作成**をクリック

### Pub/Subの設定 @Pub/Sub
1. [コンソール](https://console.cloud.google.com/cloudpubsub)へ移動
2. 左のメニューから**サブスクリプション**をクリック
3. 上部の**サブスクリプションを作成**をクリック
4. **サブスクリプションID**に任意のIDを入力(ex. yourname-subscription)
5. Cloud Pub/Sub トピックには先程作成したトピックを選択
6. **配信タイプ**は*pull*を選択
7. 他はデフォルトのまま、**作成**をクリック

### プログラムの実行 @ローカル環境 or Cloud Shell
1. 仮想環境準備
```
virtualenv env
source env/bin/activate
```
2. サンプルプログラムの取得
```
git clone https://github.com/hagigigi/gcp-iot-core-handson.git
```
3. ライブラリ等のインストール
```
cd gcp-iot-core-handson/
pip install -r requirements.txt
wget https://pki.goog/roots.pem
```
pip instal時に`ERROR: google-cloud-pubsub 1.5.0 has requirement google-api-core[grpc]<1.17.0,>=1.14.0, but you'll have google-api-core 1.17.0 which is incompatible.`と出るが無視してよい
4. 実行
```
python iot_mqtt_sample.py --project_id=(your-project-id) --cloud_region=(your-region) --registry_id=(your-registry-id) --device_id=(your-device-id) --algorithm=RS256 --private_key_file=../rsa_private.pem
```
## STEP.2 Pub/SubからBigQueryへの転送
### BiqQueryにテーブルを作成 @BigQuery
1. [コンソール](https://console.cloud.google.com/bigquery)へ移動
2. 左のメニューのリソースから自身のプロジェクトをクリック
3. 右方にある**データセットを作成**をクリック
4. **デバイスID**に任意のIDを入力(ex. yourname_dataset)
5. 他はデフォルトのまま、**作成**をクリック
6. 作成したデータセットをクリックして右方にある**テーブルを作成**をクリック
7. **ソース**は*空のテーブル*を選択、**テーブル名**に任意のIDを入力(ex. yourname_table)
8. **スキーマ**に以下を追加
|  名前  |  型  |  モード  |
| ---- | ---- | ---- |
|  data  |  FLOAT  | NULLABLE |
|  timestamp  |  STRING  | NULLABLE |

### Dataflowの設定 @Dataflow
1. [コンソール](https://console.cloud.google.com/dataflow)へ移動
2. 左のメニューのリソースから**ジョブ**をクリック
3. 上部のテンプレートから**ジョブを作成**をクリック
4. **ジョブ名**に任意のIDを入力(ex. yourname-job)
5. **リージョンエンドポイント**に IoT Core で選択したリージョンを入力(ex. us-central1)
6. **Dataflowテンプレート**から*Pub/Sub Topic to BigQuery*を選択
7. パラメータは以下の通り
   - トピックID: projects/*your-project-id*/topics/*your-event-id*
   - BigQueryテーブル名: *your-project-id*:*your-dataset-id*.*your-table-id*
   - 一時的なロケーション: gs://*your-bucket-name*/*path*
8. 他はデフォルトのまま、**ジョブを実行**をクリック

### データ格納の確認 @BigQuery
1. [コンソール](https://console.cloud.google.com/bigquery)へ移動
2. 左のメニューのリソースから自身のテーブルをクリック
3. 中央タブのプレビューをクリック
4. データが入っているか確認

## 参考
- [GCP Python サンプル集](https://github.com/GoogleCloudPlatform/python-docs-samples)
- [GCP IoT クイックスタート](https://cloud.google.com/iot/docs/quickstart)
- [MQTT について](https://www.ibm.com/developerworks/jp/iot/library/iot-mqtt-why-good-for-iot/index.html)