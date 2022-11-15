# Node-REDを活用したIoT実習

## Node-REDの開発環境をDockerで構築

### Node-REDの起動と終了

#### 起動方法

起動はターミナルから以下のコマンドを実行

```shell
docker-compose up -d
```

同一ネットワーク内からブラウザで，http://localhost:8080 でアクセスします.

#### 終了方法

終了はターミナルで以下のコマンドを実行

```shell
docker-compose down
```

### Node-Redの画面構成

<center>
  <img src="./images/node-red-capture.png" width="80%">
</center>

Node-REDは、Flowエリアにノードパレットからノードをドラッグ＆ドロップし、ノード間をつなぐことで機能を組み立てていきます。

### イメージ作成コンテナ作成時に失敗によってできる<none>を削除

イメージの`<none>`を消す方法

```shell
docker image prune
```
