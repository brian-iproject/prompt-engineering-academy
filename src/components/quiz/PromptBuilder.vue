<template>
  <div class="prompt-builder" :class="stateClass">
    <div class="prompt-builder__header">
      <span class="prompt-builder__badge">Собери промпт</span>
    </div>
    <p class="prompt-builder__question">{{ props.question }}</p>
    <p class="prompt-builder__task"><strong>Задача:</strong> {{ props.task }}</p>

    <div class="prompt-builder__workspace">
      <div class="prompt-builder__slots">
        <div
          v-for="slotId in props.slots"
          :key="slotId"
          class="prompt-builder__slot"
          :class="slotClass(slotId)"
          @dragover.prevent="dragOverSlot = slotId"
          @dragleave="dragOverSlot = null"
          @drop="onDropToSlot(slotId)"
        >
          <div class="prompt-builder__slot-label">
            <span class="prompt-builder__slot-icon">{{ slotIcon(slotId) }}</span>
            {{ slotName(slotId) }}
          </div>

          <div v-if="filled[slotId]" class="prompt-builder__filled-block" :class="`prompt-builder__filled-block--${slotId}`">
            <span>{{ blockById(filled[slotId])?.text }}</span>
            <button v-if="!answered" class="prompt-builder__remove" @click="removeFromSlot(slotId)">×</button>
          </div>
          <div v-else class="prompt-builder__drop-hint">
            Перетащи сюда
          </div>
        </div>
      </div>

      <div class="prompt-builder__blocks">
        <p class="prompt-builder__blocks-label">Доступные блоки:</p>
        <div class="prompt-builder__blocks-list">
          <div
            v-for="block in availableBlocks"
            :key="block.id"
            class="prompt-builder__block"
            :class="`prompt-builder__block--${block.type}`"
            draggable="true"
            @dragstart="draggingBlock = block.id"
            @dragend="draggingBlock = null"
          >
            {{ block.text }}
          </div>
        </div>
      </div>
    </div>

    <div v-if="!answered" class="prompt-builder__actions">
      <button
        class="prompt-builder__submit"
        :disabled="Object.keys(filled).length < props.slots.length"
        @click="submit"
      >
        Проверить сборку
      </button>
    </div>

    <div v-else class="prompt-builder__result">
      <div class="prompt-builder__verdict" :class="isCorrect ? 'prompt-builder__verdict--correct' : 'prompt-builder__verdict--partial'">
        <span v-if="isCorrect">✓ Промпт собран правильно!</span>
        <span v-else>{{ correctSlots }} из {{ props.slots.length }} блоков на месте</span>
      </div>
      <div v-if="props.explanation" class="prompt-builder__explanation">
        <strong>Пояснение:</strong> {{ props.explanation }}
      </div>
      <button class="prompt-builder__reset" @click="reset">Попробовать снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue'

const props = defineProps({
  question: { type: String, required: true },
  task: { type: String, required: true },
  blocks: { type: Array, required: true },
  slots: { type: Array, required: true },
  explanation: { type: String, default: '' }
})

const emit = defineEmits(['answer'])

const draggingBlock = ref(null)
const dragOverSlot = ref(null)
const filled = reactive({})
const answered = ref(false)

const SLOT_NAMES = { role: 'Роль', context: 'Контекст', task: 'Задача', format: 'Формат' }
const SLOT_ICONS = { role: '🎭', context: '📋', task: '🎯', format: '📐' }

function slotName(id) { return SLOT_NAMES[id] || id }
function slotIcon(id) { return SLOT_ICONS[id] || '▸' }

const usedBlockIds = computed(() => Object.values(filled))

const availableBlocks = computed(() =>
  props.blocks.filter(b => !usedBlockIds.value.includes(b.id))
)

function blockById(id) {
  return props.blocks.find(b => b.id === id)
}

function slotClass(slotId) {
  const classes = []
  if (dragOverSlot.value === slotId) classes.push('prompt-builder__slot--drag-over')
  if (answered.value && filled[slotId]) {
    const block = blockById(filled[slotId])
    classes.push(block?.type === slotId ? 'prompt-builder__slot--correct' : 'prompt-builder__slot--wrong')
  }
  return classes.join(' ')
}

function onDropToSlot(slotId) {
  dragOverSlot.value = null
  if (!draggingBlock.value) return
  const existingSlot = Object.keys(filled).find(k => filled[k] === draggingBlock.value)
  if (existingSlot) delete filled[existingSlot]
  filled[slotId] = draggingBlock.value
  draggingBlock.value = null
}

function removeFromSlot(slotId) {
  delete filled[slotId]
}

const correctSlots = computed(() =>
  props.slots.filter(slotId => {
    const blockId = filled[slotId]
    if (!blockId) return false
    return blockById(blockId)?.type === slotId
  }).length
)

const isCorrect = computed(() => correctSlots.value === props.slots.length)

const stateClass = computed(() => {
  if (!answered.value) return ''
  return isCorrect.value ? 'prompt-builder--correct' : 'prompt-builder--partial'
})

function submit() {
  answered.value = true
  emit('answer', { correct: isCorrect.value, score: correctSlots.value, maxScore: props.slots.length })
}

function reset() {
  answered.value = false
  props.slots.forEach(s => delete filled[s])
}
</script>

<style lang="scss" scoped>
.prompt-builder {
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
    margin-bottom: $space-2;
    line-height: $line-height-base;
  }

  &__task {
    font-size: $font-size-sm;
    color: $color-text-muted;
    margin-bottom: $space-5;
    background: $color-bg-elevated;
    padding: $space-3;
    border-radius: $radius-md;
  }

  &__workspace {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: $space-5;
    margin-bottom: $space-5;

    @media (max-width: 640px) {
      grid-template-columns: 1fr;
    }
  }

  &__slots {
    display: flex;
    flex-direction: column;
    gap: $space-3;
  }

  &__slot {
    border: 2px dashed $color-border;
    border-radius: $radius-md;
    padding: $space-3;
    min-height: 80px;
    transition: all $transition-fast;

    &--drag-over {
      border-color: $color-primary;
      background: $color-primary-light;
    }

    &--correct {
      border-color: $color-success;
      border-style: solid;
    }

    &--wrong {
      border-color: $color-error;
      border-style: solid;
    }
  }

  &__slot-label {
    font-size: $font-size-xs;
    font-weight: $font-weight-semibold;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: $color-text-muted;
    margin-bottom: $space-2;
    display: flex;
    align-items: center;
    gap: $space-1;
  }

  &__slot-icon { font-size: $font-size-base; }

  &__filled-block {
    background: $color-bg-elevated;
    border-radius: $radius-sm;
    padding: $space-2 $space-3;
    font-size: $font-size-xs;
    line-height: $line-height-relaxed;
    color: $color-text;
    display: flex;
    align-items: flex-start;
    gap: $space-2;

    &--role { border-left: 3px solid #a78bfa; }
    &--context { border-left: 3px solid #60a5fa; }
    &--task { border-left: 3px solid #34d399; }
    &--format { border-left: 3px solid #fb923c; }
  }

  &__remove {
    background: none;
    border: none;
    color: $color-text-faint;
    cursor: pointer;
    font-size: $font-size-lg;
    padding: 0;
    line-height: 1;
    margin-left: auto;
    flex-shrink: 0;

    &:hover { color: $color-error; }
  }

  &__drop-hint {
    font-size: $font-size-xs;
    color: $color-text-faint;
    text-align: center;
    padding: $space-3;
  }

  &__blocks-label {
    font-size: $font-size-xs;
    font-weight: $font-weight-semibold;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: $color-text-muted;
    margin-bottom: $space-3;
  }

  &__blocks-list {
    display: flex;
    flex-direction: column;
    gap: $space-2;
  }

  &__block {
    padding: $space-2 $space-3;
    border-radius: $radius-sm;
    font-size: $font-size-xs;
    line-height: $line-height-relaxed;
    cursor: grab;
    transition: all $transition-fast;
    user-select: none;
    border: 1px solid $color-border;
    background: $color-bg-elevated;
    color: $color-text;

    &:hover { border-color: $color-primary; }
    &:active { cursor: grabbing; }

    &--role { border-left: 3px solid #a78bfa; }
    &--context { border-left: 3px solid #60a5fa; }
    &--task { border-left: 3px solid #34d399; }
    &--format { border-left: 3px solid #fb923c; }
    &--noise { border-left: 3px solid $color-border; opacity: 0.7; }
  }

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
