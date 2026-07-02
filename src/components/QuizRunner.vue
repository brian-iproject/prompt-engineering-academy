<template>
  <div class="quiz-runner">
    <div v-if="!finished">
      <div class="quiz-runner__progress">
        <div class="quiz-runner__progress-info">
          <span class="quiz-runner__step">Вопрос {{ currentIndex + 1 }} из {{ questions.length }}</span>
          <span class="quiz-runner__score-live">Очки: {{ totalScore }}</span>
        </div>
        <div class="quiz-runner__bar">
          <div class="quiz-runner__bar-fill" :style="{ width: progressPercent + '%' }" />
        </div>
      </div>

      <transition name="quiz-slide" mode="out-in">
        <div :key="currentIndex" class="quiz-runner__question-wrap">
          <QuizRenderer
            :component="current.component"
            :quiz-props="current.props"
            @answer="onAnswer"
          />
        </div>
      </transition>

      <div v-if="currentAnswered" class="quiz-runner__nav">
        <button
          v-if="currentIndex < questions.length - 1"
          class="quiz-runner__next"
          @click="nextQuestion"
        >
          Следующий вопрос →
        </button>
        <button v-else class="quiz-runner__finish" @click="finish">
          Завершить тест
        </button>
      </div>
    </div>

    <div v-else class="quiz-runner__summary">
      <div class="quiz-runner__summary-icon">{{ summaryEmoji }}</div>
      <h2 class="quiz-runner__summary-title">{{ summaryTitle }}</h2>
      <p class="quiz-runner__summary-subtitle">{{ summarySubtitle }}</p>

      <div class="quiz-runner__summary-score">
        <span class="quiz-runner__score-big">{{ totalScore }}</span>
        <span class="quiz-runner__score-max">/ {{ maxPossibleScore }}</span>
      </div>

      <div class="quiz-runner__summary-stats">
        <div class="quiz-runner__stat">
          <span class="quiz-runner__stat-val">{{ correctAnswers }}</span>
          <span class="quiz-runner__stat-label">Верных ответов</span>
        </div>
        <div class="quiz-runner__stat">
          <span class="quiz-runner__stat-val">{{ questions.length - correctAnswers }}</span>
          <span class="quiz-runner__stat-label">Ошибок</span>
        </div>
        <div class="quiz-runner__stat">
          <span class="quiz-runner__stat-val">{{ Math.round((totalScore / maxPossibleScore) * 100) }}%</span>
          <span class="quiz-runner__stat-label">Результат</span>
        </div>
      </div>

      <div class="quiz-runner__summary-bar">
        <div class="quiz-runner__summary-bar-fill" :style="{ width: finalPercent + '%' }" />
      </div>

      <div class="quiz-runner__recommendations">
        <h3>Рекомендации:</h3>
        <p>{{ recommendation }}</p>
      </div>

      <button class="quiz-runner__restart" @click="restart">Пройти снова</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import QuizRenderer from './quiz/QuizRenderer.vue'

const props = defineProps({
  questions: { type: Array, required: true }
})

const currentIndex = ref(0)
const currentAnswered = ref(false)
const answers = ref([])
const finished = ref(false)

const current = computed(() => props.questions[currentIndex.value])

const progressPercent = computed(() =>
  ((currentIndex.value) / props.questions.length) * 100
)

const totalScore = computed(() => answers.value.reduce((s, a) => s + a.score, 0))
const maxPossibleScore = computed(() => answers.value.reduce((s, a) => s + a.maxScore, 0) || props.questions.length)
const correctAnswers = computed(() => answers.value.filter(a => a.correct).length)

const finalPercent = computed(() =>
  maxPossibleScore.value ? Math.round((totalScore.value / maxPossibleScore.value) * 100) : 0
)

const summaryEmoji = computed(() => {
  const p = finalPercent.value
  if (p >= 90) return '🏆'
  if (p >= 70) return '🎯'
  if (p >= 50) return '📚'
  return '💡'
})

const summaryTitle = computed(() => {
  const p = finalPercent.value
  if (p >= 90) return 'Превосходно!'
  if (p >= 70) return 'Хороший результат'
  if (p >= 50) return 'Есть над чем работать'
  return 'Нужно повторить материал'
})

const summarySubtitle = computed(() => {
  const p = finalPercent.value
  if (p >= 90) return 'Вы отлично усвоили курс по промпт-инжинирингу.'
  if (p >= 70) return 'Большинство концепций усвоено. Повторите слабые места.'
  if (p >= 50) return 'Базовые концепции понятны, но детали требуют повторения.'
  return 'Рекомендуем вернуться к статьям и пройти тест снова.'
})

const recommendation = computed(() => {
  const p = finalPercent.value
  if (p >= 90) return 'Попробуйте применять знания на реальных задачах: создайте собственную библиотеку промптов для рабочих задач.'
  if (p >= 70) return 'Обратите внимание на статью о галлюцинациях — это ключевой навык для безопасной работы с AI.'
  if (p >= 50) return 'Перечитайте статью "Анатомия эффективного промпта" и уделите внимание структуре запросов.'
  return 'Начните с основ: статья "Как работают LLM" даст необходимый фундамент для понимания остальных тем.'
})

function onAnswer(result) {
  if (currentAnswered.value) return
  answers.value.push(result)
  currentAnswered.value = true
}

function nextQuestion() {
  currentIndex.value++
  currentAnswered.value = false
}

function finish() {
  finished.value = true
}

function restart() {
  currentIndex.value = 0
  currentAnswered.value = false
  answers.value = []
  finished.value = false
}
</script>

<style lang="scss" scoped>
.quiz-runner {
  &__progress {
    margin-bottom: $space-6;
  }

  &__progress-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: $space-2;
  }

  &__step {
    font-size: $font-size-sm;
    color: $color-text-muted;
    font-weight: $font-weight-medium;
  }

  &__score-live {
    font-size: $font-size-sm;
    color: $color-primary;
    font-weight: $font-weight-semibold;
  }

  &__bar {
    height: 4px;
    background: $color-bg-elevated;
    border-radius: $radius-full;
    overflow: hidden;
  }

  &__bar-fill {
    height: 100%;
    background: linear-gradient(90deg, $color-primary, $color-primary-hover);
    border-radius: $radius-full;
    transition: width $transition-slow;
  }

  &__question-wrap {
    min-height: 200px;
  }

  &__nav {
    display: flex;
    justify-content: flex-end;
    margin-top: $space-4;
  }

  &__next,
  &__finish {
    padding: $space-3 $space-6;
    background: $color-primary;
    color: white;
    border: none;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    cursor: pointer;
    transition: all $transition-fast;

    &:hover { background: $color-primary-hover; }
  }

  &__finish {
    background: $color-success;

    &:hover { background: #16a34a; }
  }

  &__summary {
    text-align: center;
    padding: $space-8 $space-4;
  }

  &__summary-icon {
    font-size: 4rem;
    margin-bottom: $space-4;
  }

  &__summary-title {
    font-size: $font-size-3xl;
    margin-bottom: $space-2;
  }

  &__summary-subtitle {
    color: $color-text-muted;
    font-size: $font-size-lg;
    margin-bottom: $space-8;
  }

  &__summary-score {
    margin-bottom: $space-6;
  }

  &__score-big {
    font-size: 5rem;
    font-weight: $font-weight-bold;
    color: $color-primary;
    line-height: 1;
  }

  &__score-max {
    font-size: $font-size-2xl;
    color: $color-text-muted;
  }

  &__summary-stats {
    display: flex;
    justify-content: center;
    gap: $space-8;
    margin-bottom: $space-6;
  }

  &__stat {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: $space-1;
  }

  &__stat-val {
    font-size: $font-size-2xl;
    font-weight: $font-weight-bold;
    color: $color-text;
  }

  &__stat-label {
    font-size: $font-size-sm;
    color: $color-text-muted;
  }

  &__summary-bar {
    height: 8px;
    background: $color-bg-elevated;
    border-radius: $radius-full;
    overflow: hidden;
    max-width: 400px;
    margin: 0 auto $space-8;
  }

  &__summary-bar-fill {
    height: 100%;
    background: linear-gradient(90deg, $color-primary, $color-success);
    border-radius: $radius-full;
    transition: width 1s ease;
  }

  &__recommendations {
    background: $color-bg-card;
    border: 1px solid $color-border;
    border-radius: $radius-lg;
    padding: $space-5;
    text-align: left;
    max-width: 560px;
    margin: 0 auto $space-6;

    h3 {
      font-size: $font-size-base;
      margin-bottom: $space-2;
      color: $color-primary;
    }

    p {
      font-size: $font-size-sm;
      color: $color-text-muted;
      line-height: $line-height-relaxed;
    }
  }

  &__restart {
    padding: $space-3 $space-8;
    background: transparent;
    border: 2px solid $color-primary;
    color: $color-primary;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    cursor: pointer;
    transition: all $transition-fast;

    &:hover {
      background: $color-primary;
      color: white;
    }
  }
}

.quiz-slide-enter-active,
.quiz-slide-leave-active {
  transition: all $transition-base;
}

.quiz-slide-enter-from {
  opacity: 0;
  transform: translateX(20px);
}

.quiz-slide-leave-to {
  opacity: 0;
  transform: translateX(-20px);
}
</style>
