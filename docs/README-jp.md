# STOMP Tester
[旅行プランナー](https://github.com/planner-project/backend) プロジェクト遂行中、チームにWebSocketテストツールを提供したくて作りました。 一般的に使用するツールがありますが、以下のような事項を満足したかったです。
- 我々のロジックに合わせてstomp接続を要請すること
- 購読アドレスと発行アドレスを自由に修正できること
- 購読アドレスへのメッセージを視覚的に見ることができること
  - フロントがどんなメッセージを受けるのか知るために

そのようにテスター作りは始まって、デプロイをすることになりました。 🎉 <br>
関連するブログの投稿は以下でご確認ください。 <br>
- [Stompを使ってリアルタイムサービスを開発する](https://sieunnnn.oopy.io/3be9e794-ec26-4b59-979d-4e79c8dc9542)
- [websocket stomp テスター 作り with sock.js](https://sieunnnn.oopy.io/309cf08f-0cf9-4fed-97c4-6a777fa9509a)
- `予定` [Websocket Exception custom実装機](https://sieunnnn.oopy.io/c7935a1f-4346-4693-a61b-963fb745e088)

<br>
<br>

## 開始前に確認してください。
### 1. http vs https
#### https
- stomp接続時にhttpsに接続すると、vercelで配布したサイトですぐに利用できます。
- もしローカルでhttpsでテストをしたいなら[🔗 httpをhttpsに変更するための過程](#http-를-https-로-변경하기-위한-과정)を参考にしてプロジェクトを修正してください。

#### http
- vercelポリシー上、httpsをhttpに修正して入っても、再びhttpsにredirectされます。そのため、プロジェクトをクローンして実行した後、テストをしてください。

### 2. 映像で使い方を簡単に見る
クリックすると、すぐに動画に移動します。 🍀 <br>

<a href="https://www.youtube.com/watch?v=NXSc0LCAlmg"><img src="https://github.com/sieunnnn/websocketTester/assets/119668620/cf52b489-78d3-42b2-9a17-be71ea2a8ea0" width=600 /></a>

<br>
<br>

## httpをhttpsに変更するためのプロセス
> _この説明は **ウィンドウ**に基づいています。_

### 1. Keyを生成する
```
keytool -genkeypair -alias tomcat -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore keystore.p12 -validity 3650
```

### 2. application.yml設定
```yaml
server:
  port:
    ${ 使用したいポートを入力してください。 }
  ssl:
    enabled:
      true
    key-store:
      ${ Keyの位置を入力してください }
    key-store-password:
      ${SPRING_DATASOURCE_PASSWORD}
    key-store-type:
      PKCS12
    key-alias:
      tomcat
```

### 3. 接続確認
プロジェクト（Spring）の実行後、ポート番号との接続を確認します。
```
2024-06-19T12:31:42.168+09:00  INFO 5812 --- [  restartedMain] o.a.t.util.net.NioEndpoint.certificate   : Connector [https-jsse-nio-8443], TLS virtual host [_default_], certificate type [UNDEFINED] configured from keystore [C:\Users\wldsm\.keystore] using alias [tomcat] with trust store [null]
2024-06-19T12:31:42.182+09:00  INFO 5812 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8443 (https) with context path ''
2024-06-19T12:31:42.183+09:00 DEBUG 5812 --- [  restartedMain] org.springframework.web.SimpLogging      : clientOutboundChannel added SubProtocolWebSocketHandler[StompSubProtocolHandler[v10.stomp, v11.stomp, v12.stomp]]
2024-06-19T12:31:42.183+09:00 DEBUG 5812 --- [  restartedMain] org.springframework.web.SimpLogging      : clientInboundChannel added WebSocketAnnotationMethodMessageHandler[prefixes=[/pub/]]
2024-06-19T12:31:42.184+09:00  INFO 5812 --- [  restartedMain] o.s.m.s.b.SimpleBrokerMessageHandler     : Starting...
```


### 4. chrome
- Chromeに入り、アドレスバーにhttps://localhost:{port}を入力します。
- `localhostサイトに移動（安全ではない）`リンクをクリックして例外を追加します。

#### 🙏 _これは開発段階のためであり、実際のプロダクション環境では有効なSSL認証書を使用してください。_

### 5. 要請
- WebSocketのEndPointは`ws`ではなく`wss`でなければならないという点を肝に銘じてください。
- httpとポート番号を異なるように設定した場合、これに留意して要請してください。

<br>
<br>
