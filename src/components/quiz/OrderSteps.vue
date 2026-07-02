<template>
  <div class="order-steps" :class="stateClass">
    <div class="order-steps__header">
      <span class="order-steps__badge">Расставь по порядку</span>
    </div>
    <p class="order-steps__question">{{ props.question }}</p>

    <div class="order-steps__list" v-if="!answered">
      <div
        v-for="(step, i) in currentOrder"
        :key="step"
        class="order-steps__step"
        :class="{ 'order-steps__step--dragging': dragging === i, 'order-steps__step--drag-over': dragOver === i }"
        draggable="true"
        @dragstart="onDragStart(i)"
        @dragover.prevent="onDragOver(i)"
        @drop="onDrop(i)"
        @dragend="onDragEnd"
      >
        <span class="order-steps__num">{{ i + 1 }}</span>
        <span class="order-steps__drag-handle">⠿</span>
        <span class="order-steps__text">{{ props.steps[step] }}</span>
      </div>
    </div>

    <div class="order-steps__list" v-else>
      <div
        v-for="(step, i) in currentOrder"
        :key="step"
        class="order-steps__step"
        :class="stepResultClass(i, step)"
      >
        <span class="order-steps__num">{{ i + 1 }}</span>
        <span class="order-steps__icon">{{ isStepCorrect(i, step) ? '✓' : '✗' }}</span>
        <span class="order-steps__text">{{ props.steps[step] }}</span>
      </div>
    </div>

    <div v-if="!answered" class="order-steps__actions">
      <p class="order-steps__hint">Перетащите элементы в правильный порядок</p>
      <button class="order-steps__submit" @click="submit">Проверить</button>
    </div>

    <div v-else class="order-steps__result">
      <div class="order-steps__verdict" :class="isCorrect ? 'order-steps__verdict--correct' : 'order-steps__verdict--wrong'">
        <span v-if="isCorrect">✓ Порядок верный!</span>
        <span v-else>{{ correctCount }} из {{ props.steps.length }} позиций верно</span>
      </div>
      <div v-if="props.explanation" class="order-steps__explanation">
        <strong>Пояснение:</strong> {{ props.explanation }}
      </div>
      <button class="order-steps__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  question: { type: String, required: true },
  steps: { type: Array, required: true },
  correctOrder: { type: Array, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

function initOrder() {
  const arr = props.steps.map((_, i) => i)
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

const currentOrder = ref(initOrder())
const dragging = ref(null)
const dragOver = ref(null)
const answered = ref(false)

function isStepCorrect(position, stepIndex) {
  return props.correctOrder[position] === stepIndex
}

const correctCount = computed(() =>
  currentOrder.value.filter((step, i) => isStepCorrect(i, step)).length
)

const isCorrect = computed(() => correctCount.value === props.steps.length)

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isCorrect.value ? 'order-steps--correct' : 'order-steps--wrong'
})

function stepResultClass(i, step) {
  return isStepCorrect(i, step) ? 'order-steps__step--correct' : 'order-steps__step--wrong'
}

function onDragStart(i) { dragging.value = i }
function onDragOver(i) { dragOver.value = i }
function onDragEnd() { dragging.value = null; dragOver.value = null }

function onDrop(i) {
  if (dragging.value === null || dragging.value === i) return
  const arr = [...currentOrder.value]
  const [removed] = arr.splice(dragging.value, 1)
  arr.splice(i, 0, removed)
  currentOrder.value = arr
  dragging.value = null
  dragOver.value = null
}

function submit() {
  answered.value = true
  const score = correctCount.value
  emit('answer', { correct: isCorrect.value, score, maxScore: props.steps.length })
}

function reset() {
  answered.value = false
  currentOrder.value = initOrder()
}
</script>

<style lang="scss" scoped>
.order-steps {
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

  &__list {
    display: flex;
    flex-direction: column;
    gap: $space-2;
    margin-bottom: $space-5;
  }

  &__step {
    display: flex;
    align-items: center;
    gap: $space-3;
    padding: $space-3 $space-4;
    background: $color-bg-elevated;
    border: 2px solid $color-border;
    border-radius: $radius-md;
    cursor: grab;
    transition: all $transition-fast;
    user-select: none;

    &:hover { border-color: $color-primary; }

    &--dragging {
      opacity: 0.4;
      cursor: grabbing;
    }

    &--drag-over {
      border-color: $color-primary;
      background: $color-primary-light;
      transform: scale(1.01);
    }

    &--correct {
      border-color: $color-success;
      background: $color-success-light;
      cursor: default;
    }

    &--wrong {
      border-color: $color-error;
      background: $color-error-light;
      cursor: default;
    }
  }

  &__num {
    width: 28px;
    height: 28px;
    border-radius: $radius-full;
    background: $color-border;
    color: $color-text-muted;
    font-size: $font-size-sm;
    font-weight: $font-weight-bold;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  &__drag-handle {
    color: $color-text-faint;
    font-size: $font-size-lg;
    flex-shrink: 0;
  }

  &__icon {
    flex-shrink: 0;
    font-size: $font-size-lg;
    width: 24px;
    text-align: center;
  }

  &__text {
    font-size: $font-size-sm;
    line-height: $line-height-base;
    color: $color-text;
  }

  &__actions {
    display: flex;
    align-items: center;
    gap: $space-4;
    flex-wrap: wrap;
  }

  &__hint {
    font-size: $font-size-sm;
    color: $color-text-muted;
    margin: 0;
    flex: 1;
  }

  &__submit {
    padding: $space-3 $space-5;
    background: $color-primary;
    color: white;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    border: none;
    transition: all $transition-fast;

    &:hover { background: $color-primary-hover; }
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
    &--wrong { background: $color-warning-light; color: $color-warning; }
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
