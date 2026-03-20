  <script setup>

import { computed, onMounted, ref } from 'vue'
import Question from "@/Question.vue";
import he from 'he'

const amount = ref(10)
const category = ref(15)
const difficulty = ref('easy')

const url = computed(() =>
    `https://opentdb.com/api.php?amount=${amount.value}&category=${category.value}&difficulty=${difficulty.value}&type=multiple`
)
const questions = ref([]);
const currentQuestionIndex = ref(0);
const checkedAnswers = ref(false);
const correct = ref(0);
function loadQuestions(){
  fetch(url.value)
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
}
onMounted(loadQuestions);

function randomCategory(){
  category.value = Math.floor(Math.random() * 23) + 1
}

function  restartQuestion(){
  questions.value = [];
  currentQuestionIndex.value = 0;
  checkedAnswers.value = false;
  correct.value = 0;
  loadQuestions();
}

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
        <button @click="currentQuestionIndex--"
                :disabled="currentQuestionIndex === 0"
                >Prev
        </button>

        <div class="buttons">
          <button
              v-for="(question, index) in questions"
              @click="currentQuestionIndex = index"
              :disabled="currentQuestionIndex === index"
              class="question-button"
              :class="{
              'blank': !question.selected_answer,
              'selected': question.selected_answer,
              'correct': checkedAnswers && question.selected_answer === question.correct_answer,
              'incorrect': checkedAnswers && question.selected_answer !== question.correct_answer
            }"
          >
            {{ index + 1 }}</button>
        </div>

        <button @click="currentQuestionIndex++" :disabled="currentQuestionIndex === (questions.length - 1)">Next</button>
      </div>

      <hr>

      <div class="setting">
        <button @click="randomCategory">random Category</button>
        <input type="number" v-model="amount" min="1" max="50"/>
        <select v-model="difficulty">
          <option value="easy">easy</option>
          <option value="medium">medium</option>
          <option value="hard">hard</option>
        </select>
        <button @click="checkAnswers" :disabled="checkedAnswers">Check</button>
        <button @click="restartQuestion">Restart</button>
        correct: {{ correct }}
      </div>
    </main>

    <hr>
    <pre>{{ questions }}</pre>
  </template>

  <style>
  * {
    font-family: sans-serif;
  }

  h1 {
    text-align: center;
  }

  main {
    max-width: 600px;
    margin: 0 auto;
    padding: 0 16px;
  }

  button {
    padding: 6px 14px;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    background: #e0e0e0;
  }

  button:hover:not(:disabled) {
    background: #c8c8c8;
  }

  button:disabled {
    opacity: 0.4;
    cursor: default;
  }

  select, input {
    padding: 6px;
    border-radius: 6px;
    border: 1px solid #ccc;
  }

  .buttons {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 8px;
    margin-top: 16px;
  }

  .buttons {
    display: flex;
    gap: 4px;
  }

  .setting{
    display: flex;
    align-items: center;
    gap: 12px;
    margin-top: 16px;
    flex-wrap: wrap;
  }

  hr {
    margin: 16px 0;
    border: none;
    border-top: 1px solid #ddd;
  }

  .blank       { background: #f0f0f0; }
  .selected    { background: #5f9ea0; color: white; }

  .selected.correct          { background: #6abf69 !important; color: white; }
  .correct                   { background: #a8d5a2; }
  .selected.incorrect        { background: #e05c5c; color: white; }
  .question-button.incorrect { background: #e05c5c; color: white; }
  </style>