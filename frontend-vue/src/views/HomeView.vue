<template>
  <div class="mt-2">
    <div class="col-3">
      <label for="userId" class="col-form-label"> userId </label>
    </div>

    <div class="col-3">
      <input
        type="text"
        required
        class="form-control"
        placeholder="userId"
        v-model="chatMember.userId"
      />
    </div>

    <div class="col-3 mt-5">
      <label for="chatMessage" class="col-form-label"> chatMessage </label>
    </div>

    <div class="input-group mb-3">
      <input
        type="text"
        required
        class="form-control"
        placeholder="chatMessage"
        v-model="chatMember.chatMessage"
        @keyup.enter="sendMessage"
      />
      <button class="btn btn-secondary" @click="sendMessage">전송</button>
    </div>

    <div class="phone-wrap mt-5">
      <div>
        <img :src="require('@/assets/img/iphone.png')" alt="아이폰" />
      </div>
      <div class="text-wrap">
        <div v-for="(data, index) in messageArray" :key="index">
          <h3>유저이름: {{ data.userId }}</h3>
          <h3>내용: {{ data.chatMessage }}</h3>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Stomp from "webstomp-client";
import SockJS from "sockjs-client";
import constraints from "@/utils/constrains";

export default {
  data() {
    return {
      messageArray: [], // 전송한 내용을 모두 저장할 배열
      chatMember: {
        userId: "",
        chatMessage: "",
      },
      connected: false,
      stompClient: null,
    };
  },
  methods: {
    connectSocket() {
      let socket = new SockJS(constraints.SERVER_URL);                     // 벡엔드 주소로 접속
      this.stompClient = Stomp.over(socket);
      this.stompClient.connect(
        {},
        () => this.receiveMessage(),
        (e) => this.errorSocket(e)
      );
    },
    receiveMessage() {                                            
      this.connected = true;
      this.stompClient.subscribe("/sub", (response) => {                    // sub 주소로 메세지 받기
        this.messageArray.push(JSON.parse(response.body));
      });
    },
    // 접속시 에러 출력
    errorSocket(e) {
      console.log("소켓 연결 실패", e);
      this.connected = false;
    },
    sendMessage() {
      if (
        this.stompClient &&
        this.connected &&
        this.chatMember.chatMessage !== ""
      ) {
        this.stompClient.send("/pub", JSON.stringify(this.chatMember), {});  // pub 주소로 메세지 전송
        this.chatMember.chatMessage = "";
      }
    },
  },
  mounted() {
    this.connectSocket();
  },
};
</script>

<style scoped>
.phone-wrap {
  width: 20vw;

  position: relative;
}

.phone-wrap img {
  width: 100%;
}

.text-wrap {
  position: absolute;
  top: 51%;
  left: 51%;
  width: 80%;
  height: 71%;
  transform: translate(-50%, -50%);

  background-color: black;

  font-size: 20px;
  color: wheat;
  font-family: Arial, Helvetica, sans-serif;

  padding: 10px;
  overflow: scroll;
}
</style>
