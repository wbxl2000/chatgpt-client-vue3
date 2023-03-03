<script setup lang="ts">
import { ref, watchEffect } from "vue";
import axios from "axios";
import { marked } from 'marked';
import hljs from 'highlight.js';

const question = ref<string>();
const token = ref<string>("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");
const messages = ref<any[]>([{ "role": "system", "content": "You are a helpful assistant." }]);

watchEffect(() => {
  axios.defaults.headers.post['Content-Type'] = 'application/json';
  axios.defaults.headers.common['Authorization'] = `Bearer ${token.value}`;
});

marked.setOptions({
  highlight: function (code) {
    return hljs.highlightAuto(code).value;
  }
});

function send() {
  messages.value.push({
    role: "user",
    content: question.value
  });
  question.value = "";
  axios.post('https://api.openai.com/v1/chat/completions', {
    "model": "gpt-3.5-turbo",
    "messages": messages.value
  })
    .then(function (response) {
      const html = marked.parse(response.data.choices[0].message.content);
      messages.value.push({
        role: "assistant",
        content: html
      });
      console.log(response);
      console.log(html);
    })
    .catch(function (error) {
      console.log(error);
    });
}


</script>

<template>
  <div id="main">
    <div id="header">
      token <input type="text" v-model="token" />
    </div>
    <div id="chat" class="flex-center">
      <div class="answer-area">
        <div v-for="(message) in messages" v-bind:key="message">
          <div v-if="message.role === 'system'" class="message assistant" v-html="message.content"> </div>
          <div v-else-if="message.role === 'user'" class="message" v-html="message.content"> </div>
          <div v-else-if="message.role === 'assistant'" class="message assistant" v-html="message.content"> </div>
        </div>
      </div>
      <div class="input-area">
        <textarea v-model="question" @keydown.enter="send()">
            </textarea>
        <button @click="send()"> send </button>
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
  width: 100%;
  padding: 50px 0;
}

input {
  width: 400px;
}

#header {
  /* position: absolute;
  right: 20px;
  top: 20px; */
}

textarea {
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

.assistant {
  background-color: #f6f6f7;
}

#main {
  width: 100%;
  height: 100%;
  position: relative;
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
  width: 100%;
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
