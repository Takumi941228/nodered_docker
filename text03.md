# Node-REDを活用したIoT実習

## ダッシュボードを使う

### データフロー

データフローは下図となる。

<center>
  <img src="./images/dashboard.png" width="80%">
</center>

### node-red ダッシュボード機能

`node-red-dashboard` を利用することで容易にダッシュボードを作成することが可能となる．

### `node-red-dashboard` ノードの追加

パレットの管理から，ノードを追加を選択して，`node-red-dashboard` を検索し追加を行う．


### ダッシュボードの例

次の図は、Node-REDコンテナのメモリ使用率のデータをチャート表示したものである。

<center>
    <img src="./images/dashboard-1.png" width="80%">
</center>

### `node-red-contrib-os` ノードの追加

パレットの管理から，ノードを追加を選択して，`node-red-contrib-os` を検索し追加を行う．

### 各ノードの設置内容は以下

- inject
<center>
  <img src="./images/hello-world-2.png" width="80%">
</center>

- Memory
    - デフォルト

- change
    - 値の代入：`msg.topic`
        - 対象の値：(文字列)`メモリ使用率`

    - 値の代入：`msg.payload`
        - 対象の値：`masg.mayload.memusage`

<center>
    <img src="./images/dashboard-2.png" width="80%">
</center>

- chart
    - Tab：` メモリ使用率 `
    - グループ：` チャート `
    - ラベル：` メモリ使用率 `

<center>
    <img src="./images/dashboard-3.png" width="80%">
</center>

`デプロイ` ボタンをクリックしノードを有効化する

以下のURL<http://localhost:8080/ui>にアクセスする。

### （課題）全体のメモリと空き容量を追加でチャートにしよう

- change
    - 値の代入：`msg.payload`
        - 対象の値：`masg.mayload.xxxxx`

<center>
    <img src="./images/dashboard-6.png" width="80%">
</center>
