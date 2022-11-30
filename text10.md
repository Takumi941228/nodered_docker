# Node-REDを活用しIoT実習

## IoTプロトタイプを作ろう

### データフロー

データフローは下図となる。

<center>
  <img src="./images/iot.png" width="80%">
</center>

### ダッシュボードの例

次の図は，ESP32 より送信した データをダッシュボードに表示したものである。

<center>
    <img src="./images/iot-1.png" width="80%">
</center>

### IFTTTの設定

以下のアドレスにアクセス

  - <https://ifttt.com/explore>

- 手順
    - `Create`をクリック
    <center>
        <img src="./images/ifttt-1.png" width="80%">
    </center>

    - `IF This`のAddをクリックし、`webohok`を選択
    <center>
        <img src="./images/ifttt-2.png" width="80%">
    </center>

    - Receive a web requestを選択し、Event Nameを`iot_data`にし、Create triggerをクリック

    - `Then That`のAddをクリックし、`google sheetsを選択`
    - `Add row to spredsheet`を選択し、以下の図と同様にする。
    <center>
        <img src="./images/iot-3.png" width="80%">
    </center>

    - Create actionをクリック

    - `Continue`をクリック
    <center>
        <img src="./images/iot-4.png" width="80%">
    </center>

    - `Webhooks`のアイコンをクリックし、`Documentation`をクリック
    <center>
        <img src="./images/ifttt-6.png" width="80%">
    </center>

    - `{{event}}`に`iot_data`を入れ、以下のURLをコピーする。
    <center>
        <img src="./images/ifttt-7.png" width="80%">
    </center>

### 各ノードの設置内容は以下

- MQTT Broker
    - デフォルト

- mqtt in
    - server:`localhost:1883`
    - topic:`deviceXX/bme`
        - 画像では，`device01/bme`となっている。

- change
    - 名前：`気圧の取り出し（温度、湿度）`
    <center>
        <img src="./images/iot-5.png" width="80%">
    </center>

    - 名前：`URLに代入`
    <center>
        <img src="./images/iot-7.png" width="80%">
    </center>

    - 名前：`送信判定`
    <center>
        <img src="./images/iot-8.png" width="80%">
    </center>

    - 名前：`送信中`
    <center>
        <img src="./images/iot-10.png" width="80%">
    </center>

    - 名前：`停止中`
    <center>
        <img src="./images/iot-11.png" width="80%">
    </center>

- gauge
    - Tab：` データロガー `
    - グループ：` 気象情報 `
    - ラベル：` 気圧（温度、湿度） `

- chart
    - Tab：` データロガー `
    - グループ：` 気象情報 `

- delay
    - 名前：`1分ごと`
    <center>
        <img src="./images/iot-6.png" width="80%">
    </center>

- function
    - 名前：` URLを生成 `
        - コード
        ```js
        var data = msg.payload;
        var data1 = data.humid;
        var data2 = data.temp;
        var data3 = data.press;
        var payload = "https://maker.ifttt.com/trigger/iot_data/with/key/`xxxxxxxxxxx`?value1="
        payload += data1;
        payload += "&value2=";
        payload += data2;
        payload += "&value3=";
        payload += data3;
        msg.payload = payload;
        return msg;
        ```

    - 名前：`IFTフラグセット`
        - コード
        ```js
        global.set("IFT", 1);
        return msg;
        ```

    - 名前：`IFTフラグリセット`
        - コード
        ```js
        global.set("IFT", 0);
        return msg;
        ```

- http request
    - デフォルト

- button
    - Lavel：`IFTTT送信開始（IFTTT送信終了）`
    <center>
        <img src="./images/iot-9.png" width="80%">
    </center>

- text
    - Lavel：`IFTTT送信制御`
    <center>
        <img src="./images/iot-12.png" width="80%">
    </center>

- debug
    - デフォルト

`デプロイ` ボタンをクリックしノードを有効化する

以下のURL<http://localhost:8080/ui>にアクセスする。

Google Driveを確認する。

## （課題）観測データをできるだけ集めてスプレッドシートにて、グラフを作成しよう。
