# Node-REDを活用したIoT実習

## MQTTを使ってデータを取得

### MQTTとは

MQTT(Message Queue Telemetry Transport)とは、「ブローカ」と呼ばれる仲介サーバがネットワーク上に存在し、「パブリッシャ」と呼ばれる送信デバイスと「サブスクライバ」と呼ばれる受信デバイスが複数存在する。パブリッシャから送信されるデータには、「トピック」と呼ばれる区別文字列とデータが含まれている。個々のサブスクライバは、どのトピックを受信するかをブローカに登録し、常時ブローカと接続をしておきます。ブローカは、パブリッシャからデータを受信したら、そのトピックを判定して、そのトピックに登録されているサブスクライバの全てに送信します。これで、サブスクライバは特定のパブリッシャからのデータのみを受信することになる。MQTTのメッセージ構成は簡単で、高速で送受信ができるようになっていて、かつ軽量なので、IoTなどに使われる非力なプロッサでも対応でき、今日よくつかわれようになっています。

<center>
  <img src="./images/mqtt-6.png" width="80%">
</center>

### データフロー

データフローは下図となる。

<center>
  <img src="./images/mqtt.png" width="80%">
</center>

### node-red MQTT機能

`node-red-contrib-aedes` を利用することで、IoTシステムにおける軽量プロトコルのMQTT通信を行うことができる。

### `node-red-contrib-aedes` ノードの追加

パレットの管理から，ノードを追加を選択して，`node-red-contrib-aedes` を検索し追加を行う。

### MQTTの通信の仕組み

MQTT の手順で、Wi-Fiなどのネットワーク通信を用いてESP32とPC間の通信を行います。データを送信するデバイスは、ブローカと呼ばれるサーバにデータの種類を示す`topic`などと共に短い送信データを送ります。サーバは、`topic`を指定することで、受信を要望するデバイスにサーバで受信したデータのうち、要求に該当するデータを要求してきたデバイスに送信します。今回はTopicを `[device_id]/bme` とし，ESP32がpublisherとなりデータの送信を行い，node-red側のsubcriberノードを用いてデータの取得を行う．

<center>
  <img src="./images/mqtt-1.png" width="80%">
</center>

### MQTT Subscribe

MQTTサーバからの情報を処理する．Subscriberの機能を実装することでサーバーからの要求に応じたデバイスのデータ処理を行うことができます

### ダッシュボードの例

次の図は，ESP32 より送信した データをゲージ表示したものである。

<center>
    <img src="./images/mqtt-2.png" width="80%">
</center>

### 各ノードの設置内容は以下

- MQTT Broker
    - デフォルト

- mqtt in
    - server:`localhost:1883`
    - topic:`deviceXX/bme`
      - 画像では，`device01/bme`となっている。

- function

  Fuctionノードでは、msg.payloadの構造から、JSONのキーをそれぞれ`.ドット`でtemp(humid,press)を指定して、値を取り出し、`payload`に代入している。
  
  - コード

  ```js
  msg.payload = msg.payload.temp;
  return msg;
  ```
  
  ```js
  msg.payload = msg.payload.humid;
  return msg;
  ```

  ```js
  msg.payload = msg.payload.press;
  return msg;
  ```

- gauge

  Gaugeノードは、`msg.payload`の値を簡単にゲージ表示してくれる。

    - Tab：` IoTシステム `
    - グループ：` 温湿度・気圧 `
    - Lavel：` 温度（または湿度、気圧） `

- debug
    - デフォルト

### `デプロイ` ボタンをクリックしノードを有効化する

### BME280センサデータの取得

BME280センサーをESP32にI2C接続を行いデータを取得する

### ESP32とBME280との接続

|信号名|AE-BME280|ESP32 GPIO|
|:-:|:-:|:-:|
|SCL|SCK|22|
|SDA|SDI|21|
|(アドレス選択)|SDO|(3.3V)|
|(電源/VDD)|VDD|(3.3V)|
|(電源/GND)|GND|(GND)|

<center>
    <img src="./images/mqtt-3.png" width="80%">
</center>

### Arduino IDEセットアップ

### ESP32 ボードマネージャの追加

Arduino IDE内の環境設定における追加ボードマネージャに記述するアドレスは以下となります

- Arduino-ESP32 Support
  - `https://dl.espressif.com/dl/package_esp32_index.json`

ツールメニューよりボードマネージャを選択し，`esp32` で検索を行います．
検索結果から，`esp32 by Espressif Systems` のパッケージをインストールします。

### ESP32デバイスに応じたボードの選択

`ESP32 Dev Module` 等，自身が利用するデバイスに合わせてボードを選択します．

### 利用するArduino用ライブラリ

#### タイマライブラリ

- Ticker
  - ライブラリ検索で追加
  - https://github.com/sstaub/Ticker

#### BME280センサ(温度・湿度・気圧計測)ライブラリ

- SparkFun BME280 Arduino Library
  - ライブラリ検索で追加
  - https://github.com/sparkfun/SparkFun_BME280_Arduino_Library

#### MQTTプロトコルによる通信用ライブラリ

- PubSubClient
  - ライブラリ検索で追加
  - https://pubsubclient.knolleary.net/

- ArduinoJson
  - ライブラリ検索で追加
  - https://arduinojson.org

### ESP32をPublisherにして，センサ情報を送信する

以下のコードは，MQTTブローカへBME280センサで取得したデータを送信します．

```c
// ライブラリをインクルード
#include <PubSubClient.h>
#include <ArduinoJson.h>
#include <SparkFunBME280.h>

// WiFi
#include <WiFi.h>
#include <time.h>

// wifi config
#define WIFI_SSID "SSID" 
#define WIFI_PASSWORD "PASSWORD"

// MQTT config
#define MQTT_SERVER "node-redサーバのIPアドレス"
#define MQTT_PORT 1883
#define MQTT_BUFFER_SIZE 128
#define TOPIC "deviceXX/bme"

// デバイスID　デバイスIDは機器ごとにユニークにします
#define DEVICE_ID "esp001"

// BME280
BME280 bme;
BME280_SensorMeasurements measurements;

// Ticker
#include <Ticker.h>
Ticker tickerMeasure;

/* MQTT Publish */
// JSONのオブジェクトを温度、湿度、気圧用に3つの項目のため作成
const int message_capacity = JSON_OBJECT_SIZE(3);

// 静的にJSONデータを生成するためにメモリを確保
StaticJsonDocument<message_capacity> json_message;

// JSONデータを格納する文字型配列のサイズを128に設定
char message_buffer[MQTT_BUFFER_SIZE];

/* MQTT用インスタンス作成 */
// WiFiClientのクラスからこのプログラムで実際に利用するWiFiClientのオブジェクトをespClientとして作成
WiFiClient espClient;

// Clientからブローカへの通信を行うPublish、ブローカへデータの受信を要求するSubscribeの処理などの、MQTTの通信を行うためのPubsubClientのクラスから実際に処理を行うオブジェクトclientを作成
PubSubClient client(espClient);


// WiFiへの接続
void setupWiFi() {
  // connect wifi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    Serial.println(".");
    delay(100);
  }

  Serial.println("");
  Serial.print("Connected : ");
  Serial.println(WiFi.localIP());
  // sync Time
  configTime( 3600L * 9, 0, "ntp.nict.jp", "ntp.jst.mfeed.ad.jp");

  /* MQTTブローカに接続 */
  // インスタント化したオブジェクトclientの接続先のサーバを、アドレスとポート番号を設定
  client.setServer(MQTT_SERVER, MQTT_PORT);

  // 1sごとにセンサデータを取得
  tickerMeasure.attach_ms(1000, sendSensorData);

}

void sendSensorData(void) {
  //センサからデータの取得
  bme.readAllMeasurements(&measurements);

  // シリアルモニタにセンサデータを表示
  Serial.println("Humidity,Pressure,BME-Temp");
  Serial.print(measurements.humidity, 0);
  Serial.print(",");
  Serial.print(measurements.pressure / 100, 2);
  Serial.print(",");
  Serial.println(measurements.temperature, 2);
}

void setup() {
  Serial.begin(115200);

  Wire.begin();

  if (bme.beginI2C() == false) //Begin communication over I2C
  {
    Serial.println("The sensor did not respond. Please check wiring.");
    while (1); //Freeze
  }

  // WiFi接続
  setupWiFi();
}

void loop() {
  client.loop();

  // MQTT未接続の場合は，再接続
  while (!client.connected() ) {
    Serial.println("Mqtt Reconnecting");
    if ( client.connect(DEVICE_ID) ) {
      Serial.println("Mqtt Connected");
      break;
    }
  }

  /* ペイロードを作成して送信を行う．*/
  // JSONデータをクリア
  json_message.clear();

  // JSONの項目をキーと値を添えてJSONを作成
  json_message["humid"] = measurements.humidity;
  json_message["press"] = measurements.pressure / 100;
  json_message["temp"] = measurements.temperature;

  // json_messageの中のJSONデータをJSON形式の文字列message_bufferとしてシリアライズ化（文字列に変換）
  serializeJson(json_message, message_buffer, sizeof(message_buffer));

  // トピックをdevicexx/bmeして、JSON形式の文字列をパブリッシュする
  client.publish(TOPIC, message_buffer);
  delay(5000);
}
```

プログラムをコンパイル・転送を行い，シリアルモニタで起動を確認する．

`デプロイ` ボタンをクリックしノードを有効化する

以下のURL<http://localhost:8080/ui>にアクセスする。

## メッセージオブジェクトの構成

MQTT inノードに接続されたdebugノードで、msgオブジェクトの全体を表示すると以下のような構成になっている。`topic`には、`"devicexx/bmm"`という文字型データ、`payload`には、`{}`で囲まれたJSONデータで構成されている。

```json
{"topic":"device01/bme","payload":{"humid":50.56835938,"press":996.303772,"temp":20.84000015},"qos":0,"retain":false,"_topic":"device01/bme","_msgid":"ac4977df34d2bc31"}
```

さらに、Fuctionノードに接続されたdebugノードで、msgオブジェクトの全体を表示すると、`msg.payload`の中身が、Tempデータのみになっていることが確認できる。

```json
{"topic":"device01/bme","payload":20.84000015,"qos":0,"retain":false,"_topic":"device01/bme","_msgid":"4590e7ca5b4b7834"}
```

## （課題）３つのデータをチャートでも表示してみよう
