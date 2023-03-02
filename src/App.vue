<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import { marked } from 'marked';
import hljs from 'highlight.js';
// import commonmark from 'commonmark';

const question = ref<string>("");
const answer = ref<string>("empty");

const AUTH_TOKEN = "Bearer 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";

axios.defaults.headers.common['Authorization'] = AUTH_TOKEN;
axios.defaults.headers.post['Content-Type'] = 'application/json';

marked.setOptions({
  highlight: function(code) {
    return hljs.highlightAuto(code).value;
  }
});

function send() {

  var reader = new commonmark.Parser();
      var writer = new commonmark.HtmlRenderer();
      var parsed = reader.parse("\n\n当一个函数被触发多次时，防抖函数可以确保它只在最后一次触发后才被执行。下面是一个简单的防抖函数的实现：\n\n```\nfunction debounce(func, wait) {\n  let timeout;\n  return function() {\n    const context = this;\n    const args = arguments;\n    const later = function() {\n      timeout = null;\n      func.apply(context, args);\n    };\n    clearTimeout(timeout);\n    timeout = setTimeout(later, wait);\n  };\n}\n```\n\n在这个函数中，我们定义了一个 `timeout` 变量来存储 `setTimeout` 返回的 ID。当防抖函数被调用时，它会清除当前的定时器，并使用 `setTimeout` 来设置一个新的定时器，这个定时器将在指定的时间间隔后调用传入的函数。 如果在新的定时器之前，防抖函数被调用了多次，那么之前的定时器会被清除，只有最后一次调用的函数会被执行。\n\n使用方法如下：\n\n```\nfunction myFunc() {\n  console.log('Hello world!');\n}\n\nconst debouncedFunc = debounce(myFunc, 200);\n\ndebouncedFunc(); // 在200毫秒后执行\ndebouncedFunc(); // 取消上一次并在200毫秒后执行\ndebouncedFunc(); // 取消上一次并在200毫秒后执行\n```"); // parsed is a 'Node' tree
      // transform parsed if you like...
      var html = writer.render(parsed); // result is a String
      document.getElementById("answer").innerHTML = html;
  // axios.post('https://api.openai.com/v1/chat/completions', {
  //   "model": "gpt-3.5-turbo",
  //   "messages": [{ "role": "user", "content": question.value }]
  // })
  //   .then(function (response) {
  //     console.log(response);
  //     answer.value = response.data.choices[0].message.content;
  //     // const html = marked.parse(answer.value);
  //     // console.log(answer.value);
  //     // console.log(html);
  //   })
  //   .catch(function (error) {
  //     console.log(error);
  //   });
}

</script>

<template>
  <div id="div">
    <textarea id="question" v-model="question"></textarea>
    <button @click="send()"> send</button>
  </div>
  <div>
    <div id="answer"></div>
    <!-- <textarea id="answer" :value="answer"></textarea> -->
  </div>
</template>

<style scoped>
#question {
  height: 500px;
  width: 500px;
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
