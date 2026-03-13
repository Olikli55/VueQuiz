<script setup>

import { computed, onMounted, ref } from 'vue'
import Question from "@/Question.vue";
import he from 'he'

const url = 'https://opentdb.com/api.php?amount=10&category=15&difficulty=easy&type=multiple';

const questions = ref([])
const currentQuestionIndex = ref(0)
const checkedAnswers = ref(false)

onMounted(() => {
  fetch(url)
      .then((res) => res.json())
      .then((json) => {
        if (json.response_code === 0) {
          questions.value = json.results.map((question) => {
            question.question = he.decode(question.question)

            question.answers = shuffle([
              question.correct_answer,
              ...question.incorrect_answers
            ])

            return question
          })
        }
      })
})

const currentQuestion = computed(() => {
  return questions.value[currentQuestionIndex.value]
})

function shuffle(array) {
  let currentIndex = array.length;

  // While there remain elements to shuffle...
  while (currentIndex !== 0) {

    // Pick a remaining element...
    let randomIndex = Math.floor(Math.random() * currentIndex);
    currentIndex--;

    // And swap it with the current element.
    [array[currentIndex], array[randomIndex]] = [
      array[randomIndex], array[currentIndex]];
  }

  return array
}

function checkAnswers() {
  checkedAnswers.value = true
}

</script>

<template>
  <h1>Quiz!</h1>

  <main v-if="questions.length > 0">
    <Question
        :question="currentQuestion"
        :checkedAnswers="checkedAnswers"
        @selectedAnswer="(answer) => {
          currentQuestion.selected_answer = answer;
        }"
    />

    <hr>

    <div class="buttons">
      <button
          @click="currentQuestionIndex--"
          :disabled="currentQuestionIndex === 0"
      >
        Prev
      </button>

      <div class="individual-buttons">
        <button
            v-for="(question, index) in questions"
            @click="currentQuestionIndex = index"
            :disabled="currentQuestionIndex === index"
            class="question-button"
            :class="{
              'selected': question.selected_answer,
              'correct': checkedAnswers && question.selected_answer === question.correct_answer,
              'incorrect': checkedAnswers && question.selected_answer !== question.correct_answer
          }"
        >
          {{ index + 1 }}
        </button>
      </div>

      <div>
        <button
            @click="currentQuestionIndex++"
            :disabled="currentQuestionIndex === (questions.length - 1)"
        >
          Next
        </button>
        <button @click="checkAnswers()">
          Check Answers
        </button>
      </div>
    </div>

  </main>

  <hr>

  <pre>
    {{ questions }}
  </pre>
</template>

<style>

.buttons {
  display: flex;
  justify-content: space-between;
}

.individual-buttons {
  display: flex;
  gap: 4px;
}

.selected {
  background-color: cadetblue;
}

.selected.correct {
  background-color: greenyellow !important;
}

.correct {
  background-color: yellow;
}

.selected.incorrect {
  background-color: red;
}

.question-button.incorrect {
  background-color: red;
}

</style>
