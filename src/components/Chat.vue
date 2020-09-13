<template>
  <div class="chat container">
    <h2 class="center teal-text">chat</h2>
    <h6 class="grey-text text-lighten-1">Welcome {{ name }}</h6>
    <div class="card">
      <div class="card-content">
        <i class="red-text right material-icons button-pointer" v-on:click="clearChat">delete</i>
        <ul class="messages" v-chat-scroll>
          <li v-for="message in messages" :key="message.id">
            <span class="teal-text name">{{ message.name + " " }}</span>
            <span class="grey-text text-lighten-3">{{ message.content }}</span>
            <span class="grey-text time">{{ message.timestamp }}</span>
          </li>
        </ul>
      </div>

      <div class="card-action">
        <NewMessage :name="name" />
      </div>
    </div>
  </div>
</template>

<script>
import NewMessage from "@/components/NewMessage";
import { db } from "@/firebase/init";
import moment from "moment";
moment.locale("en");

export default {
  name: "Chat",
  props: ["name"],
  components: { NewMessage },
  data() {
    return { messages: [] };
  },
  methods: {
    clearChat() {
      db.collection("messages")
        .get()
        .then((res) => {
          res.forEach((element) => {
            element.ref.delete();
          });
        });
      console.log("Chat history deleted successfully");
      this.messages = [];
    },
  },
  created() {
    let ref = db.collection("messages").orderBy("timestamp");
    ref.onSnapshot((snapshot) => {
      snapshot.docChanges().forEach((change) => {
        if (change.type == "added") {
          let doc = change.doc;
          this.messages.push({
            id: doc.id,
            name: doc.data().name,
            content: doc.data().content,
            timestamp: moment(doc.data().timestamp)
              .startOf("minutes")
              .fromNow(),
          });
        }
      });
    });
  },
};
</script>

<style>
h2 {
  font-family: "Comfortaa", cursive;
}
html {
  background-color: #121212;
}
.button-pointer {
  cursor: pointer;
}
.chat h2 {
  font-size: 2.6em;
  margin-bottom: 40px;
}
.card {
  background-color: #1e1e1e;
}
.chat span {
  font-size: 1.4em;
  padding: 2px;
}
.input-text {
  color: white;
}
.chat .time {
  display: block;
  font-size: 0.9em;
}

.enter {
  font-size: 0.8rem !important;
}

.messages {
  max-height: 300px;
  overflow: auto;
}
.messages::-webkit-scrollbar {
  width: 5px;
}
.messages::-webkit-scrollbar-track {
  background: #ddd;
}
.messages::-webkit-scrollbar-thumb {
  background: #aaa;
}
</style>
