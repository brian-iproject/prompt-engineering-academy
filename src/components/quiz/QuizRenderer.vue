<template>
  <component
    :is="quizComponents[props.component]"
    v-bind="props.quizProps"
    @answer="onAnswer"
  />
</template>

<script setup>
import MultipleChoice from './MultipleChoice.vue'
import TrueFalse from './TrueFalse.vue'
import FillTheBlank from './FillTheBlank.vue'
import MatchPairs from './MatchPairs.vue'
import OrderSteps from './OrderSteps.vue'
import PromptComparator from './PromptComparator.vue'
import PromptBuilder from './PromptBuilder.vue'
import SpotTheHallucination from './SpotTheHallucination.vue'

const quizComponents = {
  MultipleChoice,
  TrueFalse,
  FillTheBlank,
  MatchPairs,
  OrderSteps,
  PromptComparator,
  PromptBuilder,
  SpotTheHallucination
}

const props = defineProps({
  component: { type: String, required: true },
  quizProps: { type: Object, default: () => ({}) }
})

const emit = defineEmits(['answer'])

function onAnswer(result) {
  emit('answer', result)
}
</script>
