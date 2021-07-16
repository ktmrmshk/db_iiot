# 製造におけるIoTデータ分析 on Databricks

オリジナル: https://github.com/tomatoTomahto/azure_databricks_iot.git

## 環境の準備

### Azure Data Lake Storage (ADLS)

Delta Laleのデータ保存のためのオブジェクトストレージ。
このNotebookではコンテナ名として`iot`を固定で使用するので、その通りに作成する。
なお、ストレージアカウント名は任意。

1. 任意の名前のADLSのストレージアカウントを作成
2. 上記のストレージアカウントの中に`iot`という名前でストレージコンテナを作成する。
3. ストレージアカウントのAccess Keyを使用してDatabricksクラスタからアクセスする。Notebook上ではSecretsを用いてAccess Keyを参照する(後述)

### Azure IoT Hub

IoTデータを受信するためのメッセージキューとして使用する。
作成したIoT Hubについて、以下の2種類のconnection stringを使ってアクセスする。

* 通常のConnection string => IoT Simulator(Raspberry Pi)からメッセージを送信する時に使用
* [Event Hub Compatible](https://devblogs.microsoft.com/iotdev/understand-different-connection-strings-in-azure-iot-hub/) connection string => DatabricksがIoT Hubからメッセージを受信する(Structured Streamingの`readStream`)際に使用する

後者のStringもNotebook上ではSecretsを使ってアクセスする。


### Azure IoT Simulator

WebベースのRaspberry Piシミュレータ: https://azure-samples.github.io/raspberry-pi-web-simulator/?lang=en

今回のデモで使用するコードは以下のgitにある。

https://github.com/tomatoTomahto/azure_databricks_iot/blob/master/Manufacturing%20IoT/iot_simulator.js

このコードの中の`const connectionString`を上記で準備したIoT HubのConnection Stringに置き換える。


### Secrets

Notebook上でADLSのAccess KeyやIoT Hubのconnection stringが平文で参照されるとリスクになる。
この問題を解決するのが、Secrets機能。Code上では平文参照できない形でこれらの値を渡すことが可能。

今回のデモでは、Secretsのkey名は以下の通り作成する。

* scope名: `iot`
  - `adls_key`: ADLSのAccess Key
  - `iothub-cs`: Event Hub Compatible connection string


Secretsは[Databricks CLI](https://github.com/databricks/databricks-cli)から設定する。

```bash
## secrets scope "iot"を作成
$ databricks secrets create-scope --scope iot

## ADLS Access Keyのsecretsを作成
$ databricks secrets put --scope iot --key adls_key --string-value 'xxxxxxxx'

## IoT Hubのsecretsを作成
$ databricks secrets put --scope iot --key iothub-cs --string-value 'xxxxxxxxxxxx'

## 確認
$ databricks secrets list --scope iot
Key name      Last updated
----------  --------------
adls_key     1625800507707
iothub-cs    1625800391691
```


