<template>
  <div class="match-pairs" :class="stateClass">
    <div class="match-pairs__header">
      <span class="match-pairs__badge">Сопоставь пары</span>
    </div>
    <p class="match-pairs__question">{{ props.question }}</p>

    <div class="match-pairs__board">
      <div class="match-pairs__column match-pairs__column--left">
        <div
          v-for="(item, i) in leftItems"
          :key="'L'+i"
          class="match-pairs__item match-pairs__item--left"
          :class="leftItemClass(i)"
          :draggable="!answered"
          @click="selectLeft(i)"
          @dragstart="onDragStart(i)"
          @dragend="onDragEnd"
        >
          <span v-if="!answered" class="match-pairs__drag-handle">⠿</span>
          <span v-if="userMatches[i] !== undefined && !answered" class="match-pairs__pair-num">{{ matchIndex(i) }}</span>
          {{ item }}
        </div>
      </div>

      <div class="match-pairs__column match-pairs__column--right">
        <div
          v-for="(item, i) in shuffledRight"
          :key="'R'+i"
          class="match-pairs__item match-pairs__item--right"
          :class="rightItemClass(shuffledRight[i], i)"
          @click="selectRight(i)"
          @dragover.prevent="dragOverRight = i"
          @dragleave="dragOverRight = null"
          @drop="onDropRight(i)"
        >
          <span v-if="matchedLeftFor(i) !== undefined && !answered" class="match-pairs__pair-num">{{ matchIndex(matchedLeftFor(i)) }}</span>
          {{ item }}
        </div>
      </div>
    </div>

    <div v-if="!answered" class="match-pairs__actions">
      <p class="match-pairs__hint">Перетащи элемент слева на пару справа — или кликни левый, затем правый.</p>
      <button
        class="match-pairs__submit"
        :disabled="Object.keys(userMatches).length < props.pairs.length"
        @click="submit"
      >
        Проверить ({{ Object.keys(userMatches).length }}/{{ props.pairs.length }})
      </button>
    </div>

    <div v-else class="match-pairs__result">
      <div class="match-pairs__verdict" :class="isAllCorrect ? 'match-pairs__verdict--correct' : 'match-pairs__verdict--partial'">
        <span v-if="isAllCorrect">✓ Все пары верны!</span>
        <span v-else>{{ correctPairs }} из {{ props.pairs.length }} пар верно</span>
      </div>
      <div v-if="props.explanation" class="match-pairs__explanation">
        <strong>Пояснение:</strong> {{ props.explanation }}
      </div>
      <button class="match-pairs__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'

const props = defineProps({
  question: { type: String, required: true },
  pairs: { type: Array, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const leftItems = computed(() => props.pairs.map(p => p.left))
const rightItems = computed(() => props.pairs.map(p => p.right))

function shuffle(arr) {
  const a = [...arr]
  for (let i = a.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

const shuffledRight = ref(shuffle(rightItems.value))
const selectedLeft = ref(null)
const draggingLeft = ref(null)
const dragOverRight = ref(null)
const userMatches = reactive({})
const answered = ref(false)

const correctPairs = computed(() =>
  Object.entries(userMatches).filter(([li, ri]) => {
    const rightVal = shuffledRight.value[ri]
    return props.pairs[Number(li)].right === rightVal
  }).length
)

const isAllCorrect = computed(() => correctPairs.value === props.pairs.length)

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isAllCorrect.value ? 'match-pairs--correct' : 'match-pairs--partial'
})

function isPairCorrect(leftIdx, rightIdx) {
  return props.pairs[leftIdx].right === shuffledRight.value[rightIdx]
}

function leftItemClass(i) {
  if (draggingLeft.value === i) return 'match-pairs__item--dragging'
  if (selectedLeft.value === i) return 'match-pairs__item--active'
  if (userMatches[i] !== undefined) {
    if (answered.value) {
      return isPairCorrect(i, userMatches[i]) ? 'match-pairs__item--correct' : 'match-pairs__item--wrong'
    }
    return 'match-pairs__item--matched'
  }
  return ''
}

function rightItemClass(text, i) {
  if (!answered.value && dragOverRight.value === i) return 'match-pairs__item--drag-over'
  const matchedLeft = Object.entries(userMatches).find(([, ri]) => ri === i)
  if (matchedLeft) {
    const li = Number(matchedLeft[0])
    if (answered.value) {
      return isPairCorrect(li, i) ? 'match-pairs__item--correct' : 'match-pairs__item--wrong'
    }
    return 'match-pairs__item--matched'
  }
  return ''
}

function selectLeft(i) {
  if (answered.value) return
  selectedLeft.value = selectedLeft.value === i ? null : i
}

function selectRight(i) {
  if (answered.value || selectedLeft.value === null) return
  userMatches[selectedLeft.value] = i
  selectedLeft.value = null
}

function matchedLeftFor(rightIdx) {
  const entry = Object.entries(userMatches).find(([, ri]) => ri === rightIdx)
  return entry ? Number(entry[0]) : undefined
}

const matchOrderMap = computed(() => {
  const map = {}
  let n = 1
  Object.keys(userMatches).forEach(li => { map[li] = n++ })
  return map
})

function matchIndex(leftIdx) {
  return matchOrderMap.value[leftIdx] ?? ''
}

function onDragStart(i) {
  draggingLeft.value = i
  selectedLeft.value = null
}

function onDragEnd() {
  draggingLeft.value = null
  dragOverRight.value = null
}

function onDropRight(ri) {
  dragOverRight.value = null
  if (draggingLeft.value === null) return
  userMatches[draggingLeft.value] = ri
  draggingLeft.value = null
}

function submit() {
  answered.value = true
  const score = correctPairs.value
  const maxScore = props.pairs.length
  emit('answer', { correct: isAllCorrect.value, score, maxScore })
}

function reset() {
  answered.value = false
  selectedLeft.value = null
  Object.keys(userMatches).forEach(k => delete userMatches[k])
  shuffledRight.value = shuffle(rightItems.value)
}
</script>

<style lang="scss" scoped>
.match-pairs {
  background: $color-bg-card;
  border: 1px solid $color-border;
  border-radius: $radius-lg;
  padding: $space-6;
  margin: $space-8 0;
  transition: border-color $transition-base;

  &--correct { border-color: $color-success; }
  &--partial { border-color: $color-warning; }

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

  &__board {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: $space-4;
    margin-bottom: $space-5;
  }

  &__column {
    display: flex;
    flex-direction: column;
    gap: $space-3;
  }

  &__drag-handle {
    color: $color-text-faint;
    margin-right: $space-2;
    flex-shrink: 0;
  }

  &__pair-num {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 20px;
    height: 20px;
    border-radius: $radius-full;
    background: $color-primary;
    color: white;
    font-size: $font-size-xs;
    font-weight: $font-weight-bold;
    flex-shrink: 0;
    margin-right: $space-2;
  }

  &__item {
    padding: $space-3 $space-4;
    background: $color-bg-elevated;
    border: 2px solid $color-border;
    border-radius: $radius-md;
    font-size: $font-size-sm;
    color: $color-text;
    cursor: pointer;
    transition: all $transition-fast;
    line-height: $line-height-base;

    &--left { cursor: grab; }
    &--left:hover { border-color: $color-primary; }
    &--right:hover { border-color: $color-primary; }

    &--dragging {
      opacity: 0.4;
      cursor: grabbing;
    }

    &--drag-over {
      border-color: $color-primary;
      background: $color-primary-light;
      transform: scale(1.01);
    }

    &--active {
      border-color: $color-primary;
      background: $color-primary-light;
      color: $color-primary;
    }

    &--matched {
      border-color: $color-primary;
      background: rgba($color-primary, 0.18);
      color: $color-text;
    }

    &--correct {
      border-color: $color-success;
      background: $color-success-light;
      color: $color-success;
    }

    &--wrong {
      border-color: $color-error;
      background: $color-error-light;
      color: $color-error;
    }
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
    white-space: nowrap;

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
    &--partial { background: $color-warning-light; color: $color-warning; }
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
