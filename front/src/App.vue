<template>
  <div id="app">
    <!-- 入室済の場合、部屋の情報を表示 -->
    <div v-if="isJoined">
      <div>{{ userName }} さん</div>
      部屋番号: {{ roomId }}
    </div>

    <!-- 未入室の場合、部屋を作る or 部屋に入るを選択 -->
    <div v-else>
      <div>名前: <input v-model="userName" type="text" /></div>

      <input type="radio" v-model="joinType" value="1" />新しく部屋を作る
      <input type="radio" v-model="joinType" value="2" />友達の部屋に入る

      <div v-if="joinType == 1">
        <input type="button" value="部屋を作る" @click="createRoom" />
      </div>

      <div v-if="joinType == 2">
        部屋番号: <input v-model="roomId" type="text" />
        <input type="button" value="入室" @click="enterRoom" />
      </div>
    </div>

    <div style="color: red">
      {{ message }}
    </div>

    <hr />

    <!-- しりとり表示 -->
    <div v-if="isJoined">
      <!-- ゲームオーバー時の表示 -->
      <div v-if="isGameOver">
        <div style="color: red">{{ posts[0].userName }} さんの負け</div>
        <input type="button" value="最初から" @click="restart" />
      </div>

      <!-- 入力欄 -->
      <div v-else>
        <div>{{ turnUserName }}さんのターン:</div>

        <input type="text" v-model="input" />
        <input type="button" value="送信" @click="postWord" />
      </div>

      <!-- 入力履歴 -->
      <div v-for="(post, i) in posts" :key="i">
        <div>↑</div>
        <div>{{ post.userName }} : " {{ post.word }} "</div>
      </div>
    </div>
  </div>
</template>

<script>
import io from "socket.io-client";

export default {
  name: "App",
  data: () => ({
    userName: "",
    joinType: 1,
    isJoined: false,
    roomId: "",
    message: "",
    input: "",
    turnUserName: "",
    posts: [],
    isGameOver: false,
    socket: io("http://localhost:3031"),
  }),

  created() {
    this.socket.on("connect", () => {
      console.log("connected");
    });
  },

  mounted() {
    this.socket.on("updateRoom", (room) => {
      this.isJoined = true;
      this.roomId = room.id;
      this.message = "";
      this.turnUserName = room.users[room.turnUserIndex].name;
      this.posts = room.posts;
      this.input = "";
      this.isGameOver = room.posts.length > 0 && room.posts[0].isGameOver;
    });

    this.socket.on("notifyError", (error) => {
      this.message = error;
    });

    this.socket.on("notifyDisconnection", (userName, turnUserName) => {
      this.message = userName + " さんが退室しました";
      this.turnUserName = turnUserName;
    });
  },

  methods: {
    createRoom() {
      this.socket.emit("create", this.userName);
      this.message = "";
    },

    enterRoom() {
      this.socket.emit("enter", this.userName, this.roomId);
      this.message = "";
    },

    postWord() {
      this.socket.emit("post", this.input);
      this.message = "";
    },

    restart() {
      this.socket.emit("restart");
      this.message = "";
    },
  },
};
</script>

<style>
#app {
  text-align: center;
}
</style>
