<template>
  <div class="fill-blank" :class="stateClass">
    <div class="fill-blank__header">
      <span class="fill-blank__badge">Заполни пропуск</span>
    </div>

    <div class="fill-blank__template">
      <template v-for="(part, i) in parsedParts" :key="i">
        <span v-if="part.type === 'text'" class="fill-blank__text">{{ part.text }}</span>
        <span v-else class="fill-blank__input-wrap">
          <input
            v-model="answers[part.id]"
            class="fill-blank__input"
            :class="inputClass(part.id)"
            :placeholder="part.hint || '...'"
            :disabled="answered"
            :style="{ width: inputWidth(part.id) }"
            @keyup.enter="submit"
          />
          <span v-if="answered" class="fill-blank__correct-val">
            {{ getCorrectAnswer(part.id) }}
          </span>
        </span>
      </template>
    </div>

    <div v-if="!answered" class="fill-blank__actions">
      <button
        class="fill-blank__submit"
        :disabled="!allFilled"
        @click="submit"
      >
        Проверить
      </button>
    </div>

    <div v-else class="fill-blank__result">
      <div class="fill-blank__verdict" :class="isCorrect ? 'fill-blank__verdict--correct' : 'fill-blank__verdict--wrong'">
        {{ isCorrect ? '✓ Верно!' : '✗ Есть ошибки' }}
      </div>
      <div v-if="props.explanation" class="fill-blank__explanation">
        <strong>Пояснение:</strong> {{ props.explanation }}
      </div>
      <button class="fill-blank__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'

const props = defineProps({
  template: { type: String, required: true },
  blanks: { type: Array, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const answers = reactive({})
const answered = ref(false)

props.blanks.forEach(b => { answers[b.id] = '' })

const parsedParts = computed(() => {
  const parts = []
  let remaining = props.template
  const blankMap = Object.fromEntries(props.blanks.map(b => [b.id, b]))

  const regex = /\{(\w+)\}/g
  let lastIndex = 0
  let match

  while ((match = regex.exec(props.template)) !== null) {
    if (match.index > lastIndex) {
      parts.push({ type: 'text', text: props.template.slice(lastIndex, match.index) })
    }
    const blankId = match[1]
    if (blankMap[blankId]) {
      parts.push({ type: 'blank', id: blankId, hint: blankMap[blankId].hint })
    }
    lastIndex = match.index + match[0].length
  }

  if (lastIndex < props.template.length) {
    parts.push({ type: 'text', text: props.template.slice(lastIndex) })
  }

  return parts
})

const allFilled = computed(() =>
  props.blanks.every(b => answers[b.id].trim() !== '')
)

function normalize(str) {
  return str.trim().toLowerCase().replace(/ё/g, 'е')
}

function isBlankCorrect(id) {
  const blank = props.blanks.find(b => b.id === id)
  if (!blank) return false
  const val = normalize(answers[id])
  if (val === normalize(blank.answer)) return true
  if (blank.alternatives) return blank.alternatives.some(a => normalize(a) === val)
  return false
}

const isCorrect = computed(() => props.blanks.every(b => isBlankCorrect(b.id)))

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isCorrect.value ? 'fill-blank--correct' : 'fill-blank--wrong'
})

function inputClass(id) {
  if (!answered.value) return ''
  return isBlankCorrect(id) ? 'fill-blank__input--correct' : 'fill-blank__input--wrong'
}

function inputWidth(id) {
  const blank = props.blanks.find(b => b.id === id)
  const len = Math.max(
    (blank?.answer?.length || 0),
    (answers[id]?.length || 0),
    (blank?.hint?.length || 3)
  )
  return `${Math.max(80, len * 10 + 20)}px`
}

function getCorrectAnswer(id) {
  return props.blanks.find(b => b.id === id)?.answer || ''
}

function submit() {
  if (!allFilled.value) return
  answered.value = true
  const score = props.blanks.filter(b => isBlankCorrect(b.id)).length
  emit('answer', { correct: isCorrect.value, score, maxScore: props.blanks.length })
}

function reset() {
  answered.value = false
  props.blanks.forEach(b => { answers[b.id] = '' })
}
</script>

<style lang="scss" scoped>
.fill-blank {
  background: $color-bg-card;
  border: 1px solid $color-border;
  border-radius: $radius-lg;
  padding: $space-6;
  margin: $space-8 0;
  transition: border-color $transition-base;

  &--correct { border-color: $color-success; }
  &--wrong { border-color: $color-error; }

  &__header { margin-bottom: $space-4; }

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

  &__template {
    font-size: $font-size-lg;
    line-height: 2.2;
    color: $color-text;
    margin-bottom: $space-5;
  }

  &__text { line-height: inherit; }

  &__input-wrap {
    display: inline-flex;
    flex-direction: column;
    align-items: center;
    vertical-align: bottom;
    position: relative;
  }

  &__input {
    background: $color-bg-elevated;
    border: 2px solid $color-border;
    border-radius: $radius-sm;
    color: $color-text;
    font-size: $font-size-base;
    padding: $space-1 $space-2;
    text-align: center;
    transition: border-color $transition-fast;
    min-width: 80px;

    &:focus { border-color: $color-primary; }

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

    &:disabled { cursor: default; }

    &::placeholder { color: $color-text-faint; }
  }

  &__correct-val {
    font-size: $font-size-xs;
    color: $color-success;
    margin-top: 2px;
  }

  &__actions { display: flex; margin-bottom: $space-4; }

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
