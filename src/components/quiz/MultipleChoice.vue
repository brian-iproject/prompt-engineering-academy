<template>
  <div class="multiple-choice" :class="stateClass">
    <div class="multiple-choice__header">
      <span class="multiple-choice__badge">{{ isMulti ? 'Несколько ответов' : 'Один ответ' }}</span>
    </div>
    <p class="multiple-choice__question">{{ props.question }}</p>

    <div class="multiple-choice__options">
      <button
        v-for="(option, i) in props.options"
        :key="i"
        class="multiple-choice__option"
        :class="optionClass(i)"
        :disabled="answered"
        @click="toggleOption(i)"
      >
        <span class="multiple-choice__marker">
          <template v-if="!answered">
            <span v-if="isMulti" class="multiple-choice__checkbox" :class="{ 'multiple-choice__checkbox--checked': isSelected(i) }"></span>
            <span v-else class="multiple-choice__radio" :class="{ 'multiple-choice__radio--checked': isSelected(i) }"></span>
          </template>
          <template v-else>
            <span v-if="isSelected(i) && isOptionCorrect(i)" class="multiple-choice__icon multiple-choice__icon--correct">✓</span>
            <span v-else-if="isSelected(i) && !isOptionCorrect(i)" class="multiple-choice__icon multiple-choice__icon--wrong">✗</span>
            <span v-else-if="!isSelected(i) && isOptionCorrect(i)" class="multiple-choice__icon multiple-choice__icon--correct">✓</span>
            <span v-else>
              <span v-if="isMulti" class="multiple-choice__checkbox"></span>
              <span v-else class="multiple-choice__radio"></span>
            </span>
          </template>
        </span>
        <span class="multiple-choice__text">{{ option }}</span>
      </button>
    </div>

    <div v-if="!answered" class="multiple-choice__actions">
      <button
        class="multiple-choice__submit"
        :disabled="selected.length === 0"
        @click="submit"
      >
        Проверить
      </button>
    </div>

    <div v-else class="multiple-choice__result">
      <div class="multiple-choice__verdict" :class="isCorrect ? 'multiple-choice__verdict--correct' : 'multiple-choice__verdict--wrong'">
        <span>{{ isCorrect ? '✓ Правильно!' : '✗ Неверно' }}</span>
        <span v-if="isMulti" class="multiple-choice__score">{{ correctCount }} / {{ props.correct.length }}</span>
      </div>
      <div v-if="props.explanation" class="multiple-choice__explanation">
        <strong>Пояснение:</strong> {{ props.explanation }}
      </div>
      <button class="multiple-choice__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  question: { type: String, required: true },
  options: { type: Array, required: true },
  correct: { type: Array, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const isMulti = computed(() => props.correct.length > 1)
const selected = ref([])
const answered = ref(false)

const correctCount = computed(() =>
  selected.value.filter(i => props.correct.includes(i)).length
)

const isCorrect = computed(() => {
  if (selected.value.length !== props.correct.length) return false
  return props.correct.every(i => selected.value.includes(i))
})

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isCorrect.value ? 'multiple-choice--correct' : 'multiple-choice--wrong'
})

function isSelected(i) { return selected.value.includes(i) }
function isOptionCorrect(i) { return props.correct.includes(i) }

function optionClass(i) {
  const sel = isSelected(i)
  if (!answered.value) return sel ? 'multiple-choice__option--selected' : ''
  if (isOptionCorrect(i)) return 'multiple-choice__option--should-be'
  if (sel && !isOptionCorrect(i)) return 'multiple-choice__option--wrong'
  return ''
}

function toggleOption(i) {
  if (answered.value) return
  if (isMulti.value) {
    const idx = selected.value.indexOf(i)
    if (idx === -1) selected.value.push(i)
    else selected.value.splice(idx, 1)
  } else {
    selected.value = [i]
  }
}

function submit() {
  answered.value = true
  const score = correctCount.value
  const maxScore = props.correct.length
  emit('answer', { correct: isCorrect.value, score, maxScore })
}

function reset() {
  answered.value = false
  selected.value = []
}
</script>

<style lang="scss" scoped>
.multiple-choice {
  background: $color-bg-card;
  border: 1px solid $color-border;
  border-radius: $radius-lg;
  padding: $space-6;
  margin: $space-8 0;
  transition: border-color $transition-base;

  &--correct { border-color: $color-success; }
  &--wrong { border-color: $color-error; }

  &__header { margin-bottom: $space-3; }

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

  &__question {
    font-size: $font-size-lg;
    color: $color-text;
    margin-bottom: $space-5;
    line-height: $line-height-base;
    white-space: pre-line;
  }

  &__options {
    display: flex;
    flex-direction: column;
    gap: $space-3;
    margin-bottom: $space-5;
  }

  &__option {
    display: flex;
    align-items: flex-start;
    gap: $space-3;
    padding: $space-3 $space-4;
    background: $color-bg-elevated;
    border: 2px solid $color-border;
    border-radius: $radius-md;
    color: $color-text;
    font-size: $font-size-base;
    text-align: left;
    transition: all $transition-fast;
    cursor: pointer;

    &:not(:disabled):hover {
      border-color: $color-primary;
      background: $color-primary-light;
    }

    &--selected {
      border-color: $color-primary;
      background: $color-primary-light;
    }

    &--should-be {
      border-color: $color-success;
      background: $color-success-light;
      color: $color-success;
    }

    &--wrong {
      border-color: $color-error;
      background: $color-error-light;
      color: $color-error;
    }

    &:disabled { cursor: default; }
  }

  &__marker {
    flex-shrink: 0;
    width: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 2px;
  }

  &__checkbox {
    width: 18px;
    height: 18px;
    border: 2px solid $color-border;
    border-radius: 4px;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all $transition-fast;
    background: transparent;

    &--checked {
      background: $color-primary;
      border-color: $color-primary;

      &::after {
        content: '';
        display: block;
        width: 5px;
        height: 9px;
        border: 2px solid white;
        border-top: none;
        border-left: none;
        transform: rotate(45deg) translateY(-1px);
      }
    }
  }

  &__radio {
    width: 18px;
    height: 18px;
    border: 2px solid $color-border;
    border-radius: 50%;
    flex-shrink: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all $transition-fast;
    background: transparent;

    &--checked {
      border-color: $color-primary;

      &::after {
        content: '';
        display: block;
        width: 8px;
        height: 8px;
        background: $color-primary;
        border-radius: 50%;
      }
    }
  }

  &__icon {
    font-size: $font-size-lg;
    font-weight: $font-weight-bold;
    line-height: 1;

    &--correct { color: $color-success; }
    &--wrong { color: $color-error; }
  }

  &__text { line-height: $line-height-base; }

  &__actions { display: flex; }

  &__submit {
    padding: $space-3 $space-6;
    background: $color-primary;
    color: white;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    transition: all $transition-fast;
    border: none;

    &:hover:not(:disabled) { background: $color-primary-hover; }

    &:disabled {
      opacity: 0.4;
      cursor: not-allowed;
    }
  }

  &__result {
    display: flex;
    flex-direction: column;
    gap: $space-4;
  }

  &__verdict {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-weight: $font-weight-semibold;
    font-size: $font-size-lg;
    padding: $space-3 $space-4;
    border-radius: $radius-md;

    &--correct { background: $color-success-light; color: $color-success; }
    &--wrong { background: $color-error-light; color: $color-error; }
  }

  &__score {
    font-size: $font-size-sm;
    opacity: 0.8;
  }

  &__explanation {
    background: $color-bg-elevated;
    border-left: 3px solid $color-primary;
    padding: $space-4;
    border-radius: 0 $radius-md $radius-md 0;
    font-size: $font-size-sm;
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

    &:hover { border-color: $color-primary; color: $color-primary; }
  }
}
</style>
