<template xmlns="http://www.w3.org/1999/html">
  <div id="app">
    <div class="title size">STOMP 테스트 해보기</div>
    <div class="size">
      <div> 프론트 팀에게 STOMP 기능을 넘겨 주기전, 테스트를 해볼 수 있습니다. <br>
        하지만 해당 테스터를 사용하기 위해서는 기능이 아래와 같은 로직을 가져야 합니다. <br>
      </div>
      <div style="margin-top: 10px">
        ✅ 웹소켓 프로토콜로 변경 되기 전, 딱 한번 prehandler 에서 토큰 인증을 시행 합니다. <br>
        <div  style="margin: 4px 0 10px 20px">
          토큰 인증을 위해 http://localhost:8080/ws?token=${connectionToken.value} api 로 요청을 합니다. <br>
          넘겨진 값을 사용 하여 토큰 인증을 하고, 문제가 없다면 모든 준비는 완료 됩니다.
        </div>
        ✅ 웹소켓 프로토콜로 변경된 후에는 토큰 인증을 하지 않습니다.
      </div>
      <div style="margin-top: 10px">
        <span>해당 테스터를 사용한 프로젝트가 궁금하다면 </span>
        <span>
          <a href="https://github.com/planner-project/backend">이곳</a>
        </span>
        <span>을 방문해주세요.</span>
      </div>
      <div style="margin-top: 10px">
        <span style="font-weight: bold">만약 도움이 되셨다면 </span>
        <span>
          <a href="" style="font-weight: bold">이곳</a>
        </span>
        <span style="font-weight: bold">에 방문하여 🌟 한번씩만 눌러 주세요.</span><br>
        <span style="font-weight: bold">개발자(취준생) 에게 큰 힘이 됩니다. 🍀</span>
      </div>
    </div>
    <div class="size">
      <n-h3 prefix="bar" style="margin: 40px 0 5px 0">
        <n-text type="primary">
          연결 하기
        </n-text>
      </n-h3>
      <div class="connect-container">
        <n-input-group-label style="margin-right: 5px">🔑 인증 토큰</n-input-group-label>
        <n-input v-model:value="connectionToken" size="small" type="text" placeholder="웹소켓 연결을 위한 토큰을 넣어주세요." :disabled="connected"/>
        <n-button :disabled="connected" n-button type="primary" @click="connect" style="margin-left: 10px">
          연결 하기
        </n-button>
      </div>
    </div>
    <div class="size">
      <n-h3 prefix="bar" style="margin: 20px 0 5px 0">
        <n-text type="primary">
          구독 하기
        </n-text>
      </n-h3>
      <div>구독 주소를 입력해 주세요.</div>
      <div class="connect-container">
        <n-input-group-label style="margin-right: 5px">🔑 구독 주소</n-input-group-label>
        <n-input v-model:value="subscriptionUrl" size="small" type="text" placeholder="구독 주소를 넣어주세요." />
        <n-button type="primary" @click="subscribe" style="margin-left: 10px">
          구독 하기
        </n-button>
      </div>
      <div>
        <n-h3 prefix="bar" style="margin: 20px 0 5px 0">
          <n-text type="primary">
            발행 하기
          </n-text>
        </n-h3>
        <div class="connect-container">
          <n-input-group-label style="margin-right: 5px">📨 발행 주소</n-input-group-label>
          <n-input v-model:value="publishDestination" size="small" type="text" placeholder="발행 주소를 넣어주세요."/>
        </div>
        <div class="connect-container" style="margin-top: 10px">
          <n-input-group-label style="margin-right: 5px">📃 DTO(JSON)</n-input-group-label>
          <n-input v-model:value="messageContent" size="small" type="textarea" placeholder="JSON 형태의 DTO 를 넣어 주세요."/>
        </div>
        <div class="size" style="display: flex; flex-direction: row; justify-content: flex-end; width: 100%">
          <n-button type="primary" @click="publish" style="width: 150px; margin-top: 10px">
            발행 하기
          </n-button>
        </div>
      </div>
      <div>
        <div style="font-size: 18px; font-weight: bold; margin: 30px 0 10px 0">📩 구독 주소로 아래와 같이 메세지가 도착 해요.</div>
        <div class="json-container">
          <div v-for="(message, index) in formattedMessages" :key="index" style="margin: 30px">
            <vue-json-pretty theme="light" :data="message" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
import SockJS from 'sockjs-client';
import { Client, Message } from '@stomp/stompjs';
import VueJsonPretty from "vue-json-pretty";
import 'vue-json-pretty/lib/styles.css';

export default defineComponent({
  components: {
    VueJsonPretty
  },
  name: 'App',
  setup() {
    const connected = ref(false);
    const client = ref<Client | null>(null);
    const subscriptionUrl = ref('');
    const publishDestination = ref('');
    const messages = ref<string[]>([]);
    const messageContent = ref('');
    const connectionToken = ref('');

    const connect = async () => {
      if (!connectionToken.value) {
        alert('웹소켓 연결을 위한 토큰이 필요해요.');
        return;
      }

      const socketUrl = `http://localhost:8080/ws?token=${connectionToken.value}`;

      try {
        const response = await fetch(socketUrl);
        if (!response.ok) {
          const error = await response.json();
          console.log(error.errorCode);
          return;
        }
      } catch (error) {
        console.error('HTTP 요청에 실패했어요.:', error);
        console.log('연결에 실패 했어요.: ' + error.message);
        return;
      }

      client.value = new Client({
        webSocketFactory: () => {
          const sock = new SockJS(socketUrl);
          sock.onclose = (event) => {
            console.error('웹소켓 연결이 끊어졌어요.:', event);
            alert('웹소켓 연결이 끊어졌어요: ' + event.reason);
          };
          return sock;
        },
        debug: (str) => {
          console.log(str);
        },
        reconnectDelay: 5000,
        heartbeatIncoming: 4000,
        heartbeatOutgoing: 4000,
      });

      client.value.onConnect = () => {
        console.log('웹소켓 연결 완료!');
        connected.value = true;
      };

      client.value.onDisconnect = () => {
        connected.value = false;
        console.log('웹소켓 연결 해제');
      };

      client.value.onStompError = (frame) => {
        console.error('Broker 가 에러를 반환 했어요.: ' + frame.headers['message']);
        console.error('상세 에러문구: ' + frame.body);
      };

      client.value.activate();
    };

    const subscribe = () => {
      if (client.value && subscriptionUrl.value) {
        client.value.subscribe(subscriptionUrl.value, (message: Message) => {
          messages.value.push(message.body);
        });
      } else {
        alert('구독 주소를 입력해주세요.');
      }
    };

    const publish = () => {
      if (client.value && publishDestination.value) {
        try {
          const messageObject = JSON.parse(messageContent.value);
          client.value.publish({
            destination: publishDestination.value,
            body: JSON.stringify(messageObject),
          });
        } catch (error) {
          alert('JSON 형태가 아니에요. 다시 한번 확인 해주세요.');
        }
      } else {
        alert('발행 주소를 입력해주세요.');
      }
    };

    const formattedMessages = computed(() => {
      return messages.value.map(message => {
        try {
          return JSON.parse(message);
        } catch (e) {
          return { error: 'JSON 형태가 아니에요. 다시 한번 확인 해주세요.' };
        }
      });
    });

    return {
      connected,
      subscriptionUrl,
      publishDestination,
      messages,
      messageContent,
      connectionToken,
      connect,
      subscribe,
      publish,
      formattedMessages
    };
  },
});
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
}

.title {
  font-family: 'GangwonEduPowerExtraBoldA';
  font-size: 35px;
  margin-top: 60px;
}

.connect-container {
  display: flex;
  flex-direction: row;
  width: 100%;
  height: auto;
}

.json-container {
  width: 100%;
  height: 550px;
  border-radius: 5px;
  background-color: #f1f3f5;
  overflow-y: scroll;
  margin-bottom: 40px;

  &::-webkit-scrollbar {
    display: none;
  }
}

.size {
  width: 90%;
  height: 100%;
}

a {
  &:visited {
    color: green;
  }
}
</style>
