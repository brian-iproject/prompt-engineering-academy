<template>
  <div class="true-false" :class="stateClass">
    <div class="true-false__header">
      <span class="true-false__badge">Верно / Неверно</span>
    </div>
    <p class="true-false__statement">{{ props.statement }}</p>

    <div v-if="!answered" class="true-false__buttons">
      <button class="true-false__btn true-false__btn--true" @click="answer(true)">
        <span class="true-false__icon">✓</span> Верно
      </button>
      <button class="true-false__btn true-false__btn--false" @click="answer(false)">
        <span class="true-false__icon">✗</span> Неверно
      </button>
    </div>

    <div v-else class="true-false__result">
      <div class="true-false__verdict" :class="isCorrect ? 'true-false__verdict--correct' : 'true-false__verdict--wrong'">
        <span class="true-false__verdict-icon">{{ isCorrect ? '✓' : '✗' }}</span>
        <span>{{ isCorrect ? 'Правильно!' : 'Неверно' }}</span>
        <span class="true-false__chosen">Вы ответили: {{ chosen ? 'Верно' : 'Неверно' }}</span>
      </div>
      <div v-if="props.explanation" class="true-false__explanation">
        <strong>Пояснение:</strong> {{ props.explanation }}
      </div>
      <button class="true-false__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  statement: { type: String, required: true },
  correct: { type: Boolean, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const answered = ref(false)
const chosen = ref(null)
const isCorrect = computed(() => chosen.value === props.correct)

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isCorrect.value ? 'true-false--correct' : 'true-false--wrong'
})

function answer(value) {
  chosen.value = value
  answered.value = true
  emit('answer', { correct: isCorrect.value, score: isCorrect.value ? 1 : 0, maxScore: 1 })
}

function reset() {
  answered.value = false
  chosen.value = null
}
</script>

<style lang="scss" scoped>
.true-false {
  background: $color-bg-card;
  border: 1px solid $color-border;
  border-radius: $radius-lg;
  padding: $space-6;
  margin: $space-8 0;
  transition: border-color $transition-base;

  &--correct { border-color: $color-success; }
  &--wrong { border-color: $color-error; }

  &__header {
    margin-bottom: $space-4;
  }

  &__badge {
    font-size: $font-size-xs;
    font-weight: $font-weight-semibold;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: $color-text-muted;
    background: $color-bg-elevated;
    padding: $space-1 $space-3;
    border-radius: $radius-full;
  }

  &__statement {
    font-size: $font-size-lg;
    color: $color-text;
    margin-bottom: $space-6;
    line-height: $line-height-base;
  }

  &__buttons {
    display: flex;
    gap: $space-4;
  }

  &__btn {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: $space-2;
    padding: $space-3 $space-6;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    transition: all $transition-base;
    background: $color-bg-elevated;
    border: 2px solid $color-border;
    color: $color-text;

    &--true:hover {
      border-color: $color-success;
      background: $color-success-light;
      color: $color-success;
    }

    &--false:hover {
      border-color: $color-error;
      background: $color-error-light;
      color: $color-error;
    }
  }

  &__icon {
    font-size: $font-size-lg;
  }

  &__result {
    display: flex;
    flex-direction: column;
    gap: $space-4;
  }

  &__verdict {
    display: flex;
    align-items: center;
    gap: $space-3;
    font-weight: $font-weight-semibold;
    font-size: $font-size-lg;
    padding: $space-3 $space-4;
    border-radius: $radius-md;

    &--correct {
      background: $color-success-light;
      color: $color-success;
    }

    &--wrong {
      background: $color-error-light;
      color: $color-error;
    }
  }

  &__verdict-icon {
    font-size: $font-size-xl;
  }

  &__chosen {
    font-size: $font-size-sm;
    font-weight: $font-weight-normal;
    margin-left: auto;
    opacity: 0.8;
  }

  &__explanation {
    background: $color-bg-elevated;
    border-left: 3px solid $color-primary;
    padding: $space-4;
    border-radius: 0 $radius-md $radius-md 0;
    font-size: $font-size-sm;
    color: $color-text;
    line-height: $line-height-relaxed;
  }

  &__reset {
    align-self: flex-start;
    background: transparent;
    border: 1px solid $color-border;
    color: $color-text-muted;
    padding: $space-2 $space-4;
    border-radius: $radius-md;
    font-size: $font-size-sm;
    transition: all $transition-fast;

    &:hover {
      border-color: $color-primary;
      color: $color-primary;
    }
  }
}
</style>
