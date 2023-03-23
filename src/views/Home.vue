<template>
  <div class="chatGPT">
    <div class="answerArea" ref="answerArea">
      <div v-for="item in allQuestionArray">
        <div class="yourQuestion" v-if="item.question">
          {{ item.question }}
        </div>
        <div class="chatGPTAnswer" style="white-space: pre-wrap;" v-if="item.answer" v-html="item.answer">
        </div>
      </div>
    </div>
    <div class="questionArea">
      <div class="questionInputDiv">
        <input ref="questionInput" v-on:keyup.enter="chatNow()" v-model="question" class="questionInput" autofocus />
      </div>
      <button @click="chatNow()" class="submitQuestionButton" :disabled="isLoading">
        <img v-if="!isLoading" src="@/assets/images/submit.svg" />
        <img v-if="isLoading" class="loadingGif" src="@/assets/images/loading.gif" />
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, watch } from "vue";
import { Configuration, OpenAIApi } from "openai";

const questionInput = ref(null);
const answerArea = ref(null);
const countNumQuestion = ref(0);
const question = ref("");
const showYourQuestion = ref("");
const answer = ref("");
const allQuestionArray = reactive([]);
const isLoading = ref(false);

const openai = new OpenAIApi(new Configuration({
  apiKey: import.meta.env.VITE_API_KEY,
}));

watch(isLoading, async (newVar, oldVar) => {
  if (!newVar) {
    setTimeout(() => {
      answerArea.value.scrollTop = answerArea.value.scrollHeight;
    }, 100);
  }
});

watch(allQuestionArray, async (newVar, oldVar) => {
  setTimeout(() => {
    answerArea.value.scrollTop = answerArea.value.scrollHeight;
  }, 100);
});

function chatNow() {
  if (!isLoading.value) {
    runChatAPI();
  }
}

function runChatAPI() {
  if (question.value !== '') {
    isLoading.value = true;

    showYourQuestion.value = question.value;
    question.value = "";

    allQuestionArray.push({question: showYourQuestion.value, answer: ""});

    openai.createChatCompletion({
      model: 'gpt-3.5-turbo',
      messages: [{ role: 'user', content: showYourQuestion.value }],
    })
    .then(res => {
      console.log(res.status)
      const stringifyAnswer = JSON.stringify(res.data.choices[0].message.content).slice(1, -1).slice(4);
      answer.value = JSON.parse(`"${stringifyAnswer}"`);
      allQuestionArray[countNumQuestion.value] = {question: showYourQuestion.value, answer: answer.value};
      countNumQuestion.value++;
      isLoading.value = false;
    });
  } else {
    alert('Please enter something~');
    questionInput.value.focus();
  }
}
</script>