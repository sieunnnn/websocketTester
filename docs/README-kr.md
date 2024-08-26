# STOMP Tester
[여행 플래너](https://github.com/planner-project/backend) 프로젝트를 수행 중, 팀원에게 웹소켓 테스트에 관한 Tool 을 제공해 주고 싶어서 만들었습니다.
일반적으로 사용하는 툴이 있지만, 아래와 같은 사항을 만족하고 싶었습니다.
- 우리의 로직에 맞게 stomp 연결 요청을 할 것
- 구독 주소와 발행 주소를 자유롭게 수정할 수 있을 것
- 구독 주소로 오는 메세지를 시각적으로 알기 쉽게 볼 수 있을 것
  - 이는 프론트가 어떤 메세지를 받는지 명확 하게 알기 위함이다.

그렇게 테스터 만들기는 시작되었고, 조금 다듬어 **배포**를 하게 되었습니다. 🎉 <br>
관련된 블로그 글은 아래에서 확인해주세요.
- [Stomp 를 사용하여 실시간 서비스 개발하기](https://sieunnnn.oopy.io/3be9e794-ec26-4b59-979d-4e79c8dc9542)
- [websocket stomp 테스터 만들기 with sock.js](https://sieunnnn.oopy.io/309cf08f-0cf9-4fed-97c4-6a777fa9509a)
- `예정` [Websocket Exception custom 구현기](https://sieunnnn.oopy.io/c7935a1f-4346-4693-a61b-963fb745e088)

<br>
<br>

## 시작 전 확인해주세요.
### 1. http vs https
#### https
- stomp 연결시 https 로 연결 한다면 vercel 로 배포한 사이트에서 바로 이용이 가능합니다.
- 만약 로컬에서 https 로 테스트를 하고 싶다면  [🔗 http-를-https-로-변경하기-위한-과정](#http-를-https-로-변경하기-위한-과정) 을 참고하여 프로젝트를 수정해주세요.

#### http
- vercel 정책상 https -> http 로 수정하여 들어가도 다시 https 로 redirect 됩니다.
- 때문에 프로젝트를 클론받아서 실행 뒤, 테스트를 하시면 됩니다.

### 2. 영상으로 사용법 쉽게 보기
클릭하면 바로 영상으로 이동합니다. 🍀 <br>

<a href="https://www.youtube.com/watch?v=NXSc0LCAlmg"><img src="https://github.com/sieunnnn/websocketTester/assets/119668620/cf52b489-78d3-42b2-9a17-be71ea2a8ea0" width=600 /></a>

<br>
<br>

## http 를 https 로 변경하기 위한 과정
> _해당 설명은 **윈도우**를 기반으로 합니다._

### 1. key 생성하기
```
keytool -genkeypair -alias tomcat -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore keystore.p12 -validity 3650
```

### 2. application.yml 설정
```yaml
server:
  port:
    ${ 사용하고자 하는 포트를 입력해주세요. }
  ssl:
    enabled:
      true
    key-store:
      ${ 키위 위치를 입력해주세요. }
    key-store-password:
      ${SPRING_DATASOURCE_PASSWORD}
    key-store-type:
      PKCS12
    key-alias:
      tomcat
```

### 3. 연결 확인
프로젝트(스프링) 실행 후, 포트 번호와 연결을 확인합니다.
```
2024-06-19T12:31:42.168+09:00  INFO 5812 --- [  restartedMain] o.a.t.util.net.NioEndpoint.certificate   : Connector [https-jsse-nio-8443], TLS virtual host [_default_], certificate type [UNDEFINED] configured from keystore [C:\Users\wldsm\.keystore] using alias [tomcat] with trust store [null]
2024-06-19T12:31:42.182+09:00  INFO 5812 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8443 (https) with context path ''
2024-06-19T12:31:42.183+09:00 DEBUG 5812 --- [  restartedMain] org.springframework.web.SimpLogging      : clientOutboundChannel added SubProtocolWebSocketHandler[StompSubProtocolHandler[v10.stomp, v11.stomp, v12.stomp]]
2024-06-19T12:31:42.183+09:00 DEBUG 5812 --- [  restartedMain] org.springframework.web.SimpLogging      : clientInboundChannel added WebSocketAnnotationMethodMessageHandler[prefixes=[/pub/]]
2024-06-19T12:31:42.184+09:00  INFO 5812 --- [  restartedMain] o.s.m.s.b.SimpleBrokerMessageHandler     : Starting...
```


### 4. chrome
- 크롬에 들어가서 주소창에 https://localhost:{port} 를 입력합니다.
- `localhost 사이트로 이동(안전하지 않음)` 링크를 클릭하여 예외를 추가합니다.

#### 🙏 _이는 개발 단계를 위함이며, 실제 프로덕션 환경에서는 유효한 SSL 인증서를 사용해주세요._

### 5. 요청하기
- 이때 웹소켓 엔드 포인트는 `ws` 가 아닌 `wss` 여야 한다는 점 명심해주세요.
- http 와 포트번호를 다르게 설정한 경우, 이에 유의하여 요청해주세요.
