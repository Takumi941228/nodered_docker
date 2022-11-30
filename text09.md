# Node-REDを活用しIoT実習

## Ambientを使う

AmbientはIoTのアイデアをなるべく簡単にプロトタイプするお手伝いをします。細かな初期設定をしなくても送ったデータをリアルタイムでグラフ化します。例えばセンサから得られた温度と湿度のデータを初期設定をしないで送った場合でも、簡単にグラフが表示されます。

### データフロー

データフローは下図となる。

<center>
    <img src="./images/ambient.png" width="80%">
</center>

### ダッシュボードの例

次の図は，ESP32 より送信した データをAmbientのダッシュボードに表示したものである。

<center>
    <img src="./images/ambient-1.png" width="80%">
</center>

### Ambientの設定

以下のアドレスにアクセス

  - <https://ambidata.io/>

- 手順
  - チャンネル一覧＞チャンネルを作る
  <center>
    <img src="./images/ambient-2.png" width="80%">
  </center>

  - 設定＞設定変更
  <center>
    <img src="./images/ambient-3.png" width="80%">
  </center>

  - チェンネル名：`ESP32温湿度気圧データロガー`
  - データ1：`温度`
  - データ2：`湿度`
  - データ3：`気圧`
  <center>
    <img src="./images/ambient-4.png" width="80%">
  </center>

  - チャンネル名`ESP32温湿度気圧データロガー`をクリック
  <center>
    <img src="./images/ambient-5.png" width="80%">
  </center>

  - チャンネルデータ設定
  <center>
    <img src="./images/ambient-6.png" width="80%">
  </center>

  - チャート名：`温湿度データ`
  - チャートの種類：`折れ線グラフ`
  - d1:温度：`左軸`
  - d2:湿度：`右軸`
  - 軸の最小最大値：`任意`

  <center>
    <img src="./images/ambient-7.png" width="80%">
  </center>

### node-red アンビエント機能

`node-red-contrib-ambient` を利用することでAmbientへのデータ送信を容易にすることができる。

### `node-red-contrib-ambient` ノードの追加

パレットの管理から，ノードを追加を選択して，`node-red-contrib-ambient` を検索し追加を行う

### 各ノードの設置内容は以下

- MQTT Broker
    - デフォルト

- mqtt in
    - server:`localhost:1883`
    - topic:`deviceXX/bme`
      - 画像では，`device01/bme`となっている。

- function
  ```js
  var data = {
    "d1": msg.payload.temp,
    "d2": msg.payload.humid,
    "d3": msg.payload.press
  }
  msg.payload = data;
  return msg;
  ```

- Ambient
    - ChannelId及びWriteKey：`自身のID及びキー`

- debug
    - デフォルト

`デプロイ` ボタンをクリックしノードを有効化する

### ESP32をPublisherにして，センサ情報を送信する

以下のコードは，MQTTブローカへBME280センサで取得したデータを送信します．

```c
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

// MQTT Publish
const int message_capacity = JSON_OBJECT_SIZE(3);
StaticJsonDocument<message_capacity> json_message;
char message_buffer[MQTT_BUFFER_SIZE];

// MQTT用インスタンス作成
WiFiClient espClient;
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

  // MQTTブローカに接続
  client.setServer(MQTT_SERVER, MQTT_PORT);

  // 1sごとにセンサデータを送信する
  tickerMeasure.attach_ms(1000, sendSensorData);

}

void sendSensorData(void) {
  //センサからデータの取得
  bme.readAllMeasurements(&measurements);
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

  // ペイロードを作成して送信を行う．
  json_message.clear();
  json_message["humid"] = measurements.humidity;
  json_message["press"] = measurements.pressure / 100;
  json_message["temp"] = measurements.temperature;
  serializeJson(json_message, message_buffer, sizeof(message_buffer));
  client.publish(TOPIC, message_buffer);
  delay(5000);
}
```

プログラムをコンパイル・転送を行い，シリアルモニタで起動を確認する．

Ambientのダッシュボードにアクセスし、データが送信されていることを確認する。

## （課題）気圧データもグラフにしてみよう
