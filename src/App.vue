<script setup lang="ts">
import { onMounted, ref, watchEffect } from "vue";
import axios from "axios";
import { marked } from 'marked';
import hljs from 'highlight.js';

const question = ref<string>();
const token = ref<string>("");
// const messages = ref<any[]>([{ "role": "system", "content": "You are a helpful assistant." }]);
const messages = ref<any[]>([]);
const areaElement = ref();
const isLock = ref<boolean>(false);

onMounted(() => {
  axios.defaults.headers.post['Content-Type'] = 'application/json';
  token.value = localStorage.getItem("token") ?? "";
  isLock.value = localStorage.getItem("lock") === "locked" ? true : false;
  if (isLock.value) {
    axios.defaults.headers.common['Authorization'] = `Bearer ${token.value}`;
  }
})

function lock() {
  isLock.value = !isLock.value;
  localStorage.setItem("lock", isLock.value ? "locked" : "unlocked");
  if (isLock.value){
    localStorage.setItem("token", token.value);
    axios.defaults.headers.common['Authorization'] = `Bearer ${token.value}`;
  } else {
    axios.defaults.headers.common['Authorization'] = "";
  }
}

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
      alert(error);
    });
}

function auto_grow() {
  areaElement.value.style.height = "5px";
  areaElement.value.style.height = (areaElement.value.scrollHeight) + "px";
}

</script>

<template>
  <div id="main">
    <div id="header">
      <input type="text" v-model="token" :disabled="isLock"/> 
      <button id="lock" @click="lock()"> {{ isLock ? "Unlock" : "Lock" }}  Secret Key </button>
      <span id="guide" v-show="!isLock"> <a href="https://platform.openai.com/account/api-keys"> how to get?</a></span>
    </div>
    <div id="chat" class="flex-center">
      <div class="answer-area">
        <template v-for="(message) in messages" v-bind:key="message">
          <div v-if="message.role === 'system'" class="message-container assistant">
            <div v-html="'[preset] ' + message.content" class="message"></div>
          </div>
          <div v-else-if="message.role === 'user'" class="message-container"> 
            <div v-html="message.content" class="message"></div>
          </div>
          <div v-else-if="message.role === 'assistant'" class="message-container assistant"> 
            <div v-html="message.content" class="message"></div>
          </div>
        </template>
      </div>
      <div class="input-area">
        <textarea ref="areaElement" v-model="question" @input="auto_grow()"> </textarea>
        <button id="send" @click="send()"> send </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.input-area {
  display: flex;
  align-items: center;
  width: 800px;
  height: auto;
  margin: 20px 0;
}

.answer-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.message-container {
  width: 100%;
  padding: 50px 300px;
  display: flex;
  justify-content: center;
}

.message {
  width: 800px;
}

input {
  width: 400px;
}

#guide {
  color: grey;
  margin: 0 5px;
}

#lock {
  margin-left: 5px;
}

#header {
  width: 100%;
  display: flex;
  justify-content: end;
  padding: 5px;
}

textarea {
  width: 750px;
  border-radius: 5px;
  border: 1px silver solid;
  padding: 8px;
  resize: none;
  scroll-padding-block: 8px;
  min-height: 50px;
  /* max-height: 100px; */
}

#send {
  height: 50px;
  margin-left: 10px;
  border-radius: 5px;
  border: 1px silver solid;
  transition: border linear 0.1s;
}

#send:hover {
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
