<script setup>

const props = defineProps({
  question: Object,
  checkedAnswers: Boolean
})

const emit = defineEmits([
    'selectedAnswer'
])



import { watch } from 'vue'

watch(() => props.checkedAnswers, (val) => {
  console.log(props.checkedAnswers)
  if (val && props.question.selected_answer === props.question.correct_answer) {
    correct++ // your global var
  }
})


</script>

<template>
  <h2> {{ question.question }} </h2>

  <div class="answers">
    <button
        v-for="(answer, index) in question.answers"
        :key="index"
        @click="emit('selectedAnswer', answer)"
        :class="{
          'blank':  checkedAnswers === false,
          'selected': answer === question.selected_answer,
          'correct': checkedAnswers && answer === question.correct_answer,
          'incorrect': checkedAnswers && answer !== question.correct_answer
        }"
        :disabled="checkedAnswers"
    >
      <span>
        <template v-if="index === 0">
          A
        </template>
        <template v-else-if="index === 1">
          B
        </template>
        <template v-else-if="index === 2">
          C
        </template>
        <template v-else>
          D
        </template>
        )
      </span>

      {{ answer }}
    </button>
  </div>
</template>

<style scoped>

h2 {
  height: 60px;
}

.answers {
  display: flex;
  flex-direction: column;
  max-width: 400px;
  height: 120px;
  gap: 8px;
}

.answers button {
  text-align: start;
}

</style>