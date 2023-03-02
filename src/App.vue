<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import { marked } from 'marked';
import hljs from 'highlight.js';

const question = ref<string>("");
const answer = ref<string>("empty");

const AUTH_TOKEN = "Bearer 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";

axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/json';

marked.setOptions({
  highlight: function (code) {
    return hljs.highlightAuto(code).value;
  }
});

const messages = ref<any[]>([]);

function send() {
  const theQuestion = question.value;
  messages.value.push(theQuestion);
  question.value = "";
  axios.post('https://api.openai.com/v1/chat/completions', {
    "model": "gpt-3.5-turbo",
    "messages": [{ "role": "user", "content": theQuestion }]
  })
    .then(function (response) {
      answer.value = response.data.choices[0].message.content;
      const html = marked.parse(answer.value);
      messages.value.push(html);
      console.log(response);
      console.log(answer.value);
      console.log(html);
    })
    .catch(function (error) {
      console.log(error);
    });
}


</script>

<template>
  <div id="main">
    <div id="sidebar" v-if="false"> New Chat
      <div></div>
    </div>
    <div id="chat" class="flex-center">
      <div class="answer-area">
        <div v-for="(message, index) in messages" v-bind:key="message">
          <div v-if="!(index%2)" class="message" v-html="message"> 
          </div>
          <div v-if="index%2" class="message ai-message" v-html="message"> 
          </div>
        </div>
      </div>
      <div class="input-area">
        <input v-model="question"> <button @click="send()"> send</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.input-area {
  display: flex;
  align-items: center;
  margin: 20px 0;
}

.answer-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.message {
  width: 750px;
  padding: 50px 0;
}

input {
  width: 750px;
  height: 50px;
  border-radius: 5px;
  border: 1px silver solid;
}

button {
  height: 50px;
  margin: 10px;
  border-radius: 5px;
  border: 1px silver solid;
  transition: border linear 0.1s;
}

button:hover {
  border: 1px rgb(118, 118, 118) solid;
}

.ai-message {
  background-color: #f6f6f7;
}

#main {
  width: 100%;
  height: 100%;
}

#sidebar {
  width: 30%;
  height: 100vh;
  color: white;
  background-color: #1d1e20;
}

.flex-center {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0 300px;
}

#answer {
  height: 1000px;
  width: 500px;
}

#div {
  display: flex;
  flex-direction: column;
}
</style>
