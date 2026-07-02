<template>
  <div class="spot-hall" :class="stateClass">
    <div class="spot-hall__header">
      <span class="spot-hall__badge">Найди галлюцинацию</span>
    </div>
    <p class="spot-hall__question">{{ props.question }}</p>

    <div v-if="!answered" class="spot-hall__hint">
      Кликни на фрагменты текста, которые кажутся тебе ложными. Выбранные фрагменты подсветятся.
    </div>

    <div class="spot-hall__text">
      <span
        v-for="(segment, i) in props.segments"
        :key="i"
        class="spot-hall__segment"
        :class="segmentClass(i)"
        @click="!answered && toggle(i)"
      >{{ segment.text }}</span>
    </div>

    <div v-if="!answered" class="spot-hall__actions">
      <span class="spot-hall__count">Отмечено: {{ selected.size }}</span>
      <button
        class="spot-hall__submit"
        :disabled="selected.size === 0"
        @click="submit"
      >
        Проверить
      </button>
    </div>

    <div v-else class="spot-hall__result">
      <div class="spot-hall__legend">
        <span class="spot-hall__legend-item spot-hall__legend-item--tp">✓ Найдена</span>
        <span class="spot-hall__legend-item spot-hall__legend-item--fn">✗ Пропущена</span>
        <span class="spot-hall__legend-item spot-hall__legend-item--fp">⚠ Лишняя отметка</span>
      </div>
      <div class="spot-hall__verdict" :class="isFullyCorrect ? 'spot-hall__verdict--correct' : 'spot-hall__verdict--partial'">
        <span v-if="isFullyCorrect">✓ Все галлюцинации найдены!</span>
        <span v-else>{{ foundCount }} из {{ totalHallucinations }} галлюцинаций найдено
          <template v-if="falsePosCount > 0"> · {{ falsePosCount }} лишних отметок</template>
        </span>
      </div>
      <div v-if="props.explanation" class="spot-hall__explanation">
        <strong>Разбор:</strong> {{ props.explanation }}
      </div>
      <button class="spot-hall__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  question: { type: String, required: true },
  segments: { type: Array, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const selected = ref(new Set())
const answered = ref(false)

const totalHallucinations = computed(() =>
  props.segments.filter(s => s.isHallucination).length
)

const foundCount = computed(() =>
  [...selected.value].filter(i => props.segments[i]?.isHallucination).length
)

const falsePosCount = computed(() =>
  [...selected.value].filter(i => !props.segments[i]?.isHallucination).length
)

const isFullyCorrect = computed(() =>
  foundCount.value === totalHallucinations.value && falsePosCount.value === 0
)

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isFullyCorrect.value ? 'spot-hall--correct' : 'spot-hall--partial'
})

function segmentClass(i) {
  const seg = props.segments[i]
  const isSelected = selected.value.has(i)

  if (!answered.value) {
    return [
      seg.isHallucination ? 'spot-hall__segment--clickable' : 'spot-hall__segment--clickable',
      isSelected ? 'spot-hall__segment--selected' : ''
    ].join(' ')
  }

  if (seg.isHallucination && isSelected) return 'spot-hall__segment--true-positive'
  if (seg.isHallucination && !isSelected) return 'spot-hall__segment--false-negative'
  if (!seg.isHallucination && isSelected) return 'spot-hall__segment--false-positive'
  return ''
}

function toggle(i) {
  const next = new Set(selected.value)
  if (next.has(i)) next.delete(i)
  else next.add(i)
  selected.value = next
}

function submit() {
  answered.value = true
  const score = Math.max(0, foundCount.value - falsePosCount.value)
  emit('answer', { correct: isFullyCorrect.value, score, maxScore: totalHallucinations.value })
}

function reset() {
  answered.value = false
  selected.value = new Set()
}
</script>

<style lang="scss" scoped>
.spot-hall {
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
    margin-bottom: $space-3;
    line-height: $line-height-base;
  }

  &__hint {
    font-size: $font-size-sm;
    color: $color-text-muted;
    background: $color-bg-elevated;
    border-radius: $radius-md;
    padding: $space-3 $space-4;
    margin-bottom: $space-4;
    border-left: 3px solid $color-warning;
  }

  &__text {
    font-size: $font-size-base;
    line-height: 2;
    background: $color-bg-elevated;
    border-radius: $radius-md;
    padding: $space-4 $space-5;
    margin-bottom: $space-5;
    font-style: italic;
    color: $color-text-muted;
  }

  &__segment {
    transition: all $transition-fast;
    border-radius: 3px;
    cursor: pointer;

    &--clickable:hover {
      background: rgba($color-warning, 0.2);
      color: $color-text;
    }

    &--selected {
      background: rgba($color-warning, 0.35);
      color: $color-text;
      text-decoration: underline;
      text-decoration-style: wavy;
      text-decoration-color: $color-warning;
    }

    &--true-positive {
      background: $color-success-light;
      color: $color-success;
      cursor: default;
    }

    &--false-negative {
      background: $color-error-light;
      color: $color-error;
      cursor: default;
      text-decoration: underline;
      text-decoration-style: wavy;
    }

    &--false-positive {
      background: $color-warning-light;
      color: $color-warning;
      cursor: default;
    }
  }

  &__actions {
    display: flex;
    align-items: center;
    gap: $space-4;
  }

  &__count {
    font-size: $font-size-sm;
    color: $color-text-muted;
    flex: 1;
  }

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

  &__legend {
    display: flex;
    gap: $space-4;
    flex-wrap: wrap;
  }

  &__legend-item {
    font-size: $font-size-xs;
    padding: 2px 8px;
    border-radius: $radius-full;

    &--tp { background: $color-success-light; color: $color-success; }
    &--fn { background: $color-error-light; color: $color-error; }
    &--fp { background: $color-warning-light; color: $color-warning; }
  }

  &__verdict {
    font-weight: $font-weight-semibold;
    font-size: $font-size-base;
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
