<template>
  <div class="prompt-comparator" :class="stateClass">
    <div class="prompt-comparator__header">
      <span class="prompt-comparator__badge">Сравни промпты</span>
    </div>
    <p class="prompt-comparator__question">{{ props.question }}</p>

    <div class="prompt-comparator__grid">
      <div
        v-for="opt in ['A', 'B']"
        :key="opt"
        class="prompt-comparator__card"
        :class="cardClass(opt)"
        @click="!answered && select(opt)"
      >
        <div class="prompt-comparator__card-header">
          <span class="prompt-comparator__label">{{ opt === 'A' ? props.promptA.label : props.promptB.label }}</span>
          <span v-if="answered" class="prompt-comparator__card-verdict">
            {{ (opt === 'A' ? props.promptA.quality : props.promptB.quality) === 'good' ? '✓ Лучше' : '✗ Хуже' }}
          </span>
        </div>

        <div class="prompt-comparator__prompt-box">
          <p class="prompt-comparator__prompt-text">{{ opt === 'A' ? props.promptA.text : props.promptB.text }}</p>
        </div>

        <div class="prompt-comparator__arrow">↓ Ответ AI</div>

        <div class="prompt-comparator__response-box">
          <p class="prompt-comparator__response-text">{{ opt === 'A' ? props.promptA.response : props.promptB.response }}</p>
        </div>

        <div v-if="!answered" class="prompt-comparator__select-btn">
          <span :class="{ 'prompt-comparator__radio--selected': chosen === opt }">
            {{ chosen === opt ? '◉' : '○' }}
          </span>
          Это лучше
        </div>
      </div>
    </div>

    <div v-if="!answered" class="prompt-comparator__actions">
      <button
        class="prompt-comparator__submit"
        :disabled="!chosen"
        @click="submit"
      >
        Подтвердить выбор
      </button>
    </div>

    <div v-else class="prompt-comparator__result">
      <div class="prompt-comparator__verdict" :class="isCorrect ? 'prompt-comparator__verdict--correct' : 'prompt-comparator__verdict--wrong'">
        {{ isCorrect ? '✓ Верно! Вы выбрали лучший промпт.' : '✗ Не совсем. Попробуйте разобраться, почему.' }}
      </div>
      <div v-if="props.explanation" class="prompt-comparator__explanation">
        <strong>Разбор:</strong> {{ props.explanation }}
      </div>
      <button class="prompt-comparator__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  question: { type: String, required: true },
  promptA: { type: Object, required: true },
  promptB: { type: Object, required: true },
  correctChoice: { type: String, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const chosen = ref(null)
const answered = ref(false)
const isCorrect = computed(() => chosen.value === props.correctChoice)

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isCorrect.value ? 'prompt-comparator--correct' : 'prompt-comparator--wrong'
})

function cardClass(opt) {
  if (answered.value) {
    const quality = opt === 'A' ? props.promptA.quality : props.promptB.quality
    return quality === 'good' ? 'prompt-comparator__card--good' : 'prompt-comparator__card--bad'
  }
  return chosen.value === opt ? 'prompt-comparator__card--selected' : ''
}

function select(opt) { chosen.value = opt }

function submit() {
  answered.value = true
  emit('answer', { correct: isCorrect.value, score: isCorrect.value ? 1 : 0, maxScore: 1 })
}

function reset() {
  answered.value = false
  chosen.value = null
}
</script>

<style lang="scss" scoped>
.prompt-comparator {
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
    margin-bottom: $space-5;
    line-height: $line-height-base;
  }

  &__grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: $space-4;
    margin-bottom: $space-5;

    @media (max-width: 600px) {
      grid-template-columns: 1fr;
    }
  }

  &__card {
    border: 2px solid $color-border;
    border-radius: $radius-md;
    padding: $space-4;
    cursor: pointer;
    transition: all $transition-base;
    display: flex;
    flex-direction: column;
    gap: $space-3;

    &:hover:not([class*='--good']):not([class*='--bad']) {
      border-color: $color-primary;
    }

    &--selected {
      border-color: $color-primary;
      background: $color-primary-light;
    }

    &--good {
      border-color: $color-success;
      background: $color-success-light;
      cursor: default;
    }

    &--bad {
      border-color: $color-error;
      background: $color-error-light;
      cursor: default;
    }
  }

  &__card-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  &__label {
    font-weight: $font-weight-bold;
    font-size: $font-size-base;
  }

  &__card-verdict {
    font-size: $font-size-sm;
    font-weight: $font-weight-semibold;
  }

  &__prompt-box {
    background: $color-bg;
    border: 1px solid $color-border;
    border-radius: $radius-sm;
    padding: $space-3;
  }

  &__prompt-text {
    font-family: $font-family-mono;
    font-size: $font-size-xs;
    color: $color-text;
    margin: 0;
    line-height: $line-height-relaxed;
    white-space: pre-wrap;
  }

  &__arrow {
    text-align: center;
    color: $color-text-faint;
    font-size: $font-size-xs;
  }

  &__response-box {
    background: $color-bg-elevated;
    border-radius: $radius-sm;
    padding: $space-3;
    flex: 1;
  }

  &__response-text {
    font-size: $font-size-xs;
    color: $color-text-muted;
    margin: 0;
    line-height: $line-height-relaxed;
    font-style: italic;
  }

  &__select-btn {
    display: flex;
    align-items: center;
    gap: $space-2;
    font-size: $font-size-sm;
    color: $color-text-muted;
    margin-top: auto;
    padding-top: $space-2;
    border-top: 1px solid $color-border;
  }

  &__radio--selected { color: $color-primary; }

  &__actions { display: flex; }

  &__submit {
    padding: $space-3 $space-6;
    background: $color-primary;
    color: white;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    border: none;
    transition: all $transition-fast;

    &:hover:not(:disabled) { background: $color-primary-hover; }
    &:disabled { opacity: 0.4; cursor: not-allowed; }
  }

  &__result {
    display: flex;
    flex-direction: column;
    gap: $space-4;
  }

  &__verdict {
    font-weight: $font-weight-semibold;
    font-size: $font-size-lg;
    padding: $space-3 $space-4;
    border-radius: $radius-md;

    &--correct { background: $color-success-light; color: $color-success; }
    &--wrong { background: $color-error-light; color: $color-error; }
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
