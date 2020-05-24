# GCP IoT Core ハンズオン

## セットアップ
仮想環境準備
```
virtualenv env
source env/bin/activate
```

ライブラリ等のインストール
```
pip install -r requirements.txt
wget https://pki.goog/roots.pem
```

## 証明書の作成
```
openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes -out rsa_cert.pem -subj "/CN=unused"
```

## GCP設定
[クイックスタート参照](https://cloud.google.com/iot/docs/quickstart)

## 実行
```
python iot_mqtt_sample.py --project_id=(your-project-id) --cloud_region=(your-region) --registry_id=(your-registry-id) --device_id=(your-device-id) --algorithm=RS256 --private_key_file=./rsa_private.pem
```



## 参考
- [GCP Python サンプル集](https://github.com/GoogleCloudPlatform/python-docs-samples)
- [MQTT について](https://www.ibm.com/developerworks/jp/iot/library/iot-mqtt-why-good-for-iot/index.html)