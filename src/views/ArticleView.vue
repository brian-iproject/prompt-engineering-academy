<template>
  <div class="article-view">
    <div v-if="!article" class="article-view__not-found container">
      <h1>Статья не найдена</h1>
      <RouterLink to="/">← Вернуться на главную</RouterLink>
    </div>

    <template v-else>
      <div class="article-view__hero">
        <div class="container">
          <RouterLink to="/" class="article-view__back">← Все статьи</RouterLink>
          <span class="article-view__tag">{{ article.tag }}</span>
          <h1 class="article-view__title">{{ article.title }}</h1>
          <p class="article-view__desc">{{ article.description }}</p>
          <div class="article-view__meta">
            <span>{{ article.readTime }} мин чтения</span>
            <span>{{ quizCount }} интерактивных упражнений</span>
          </div>
        </div>
      </div>

      <div v-if="quizCount > 0" class="article-view__progress-bar-wrap">
        <div class="container">
          <div class="article-view__progress-info">
            <span>Упражнения: {{ answeredCount }} / {{ quizCount }}</span>
            <span v-if="answeredCount === quizCount" class="article-view__progress-done">✓ Все выполнены!</span>
          </div>
          <div class="article-view__progress-track">
            <div
              class="article-view__progress-fill"
              :style="{ width: progressPct + '%' }"
            />
          </div>
        </div>
      </div>

      <div class="article-view__body container">
        <template v-for="(section, i) in article.sections" :key="i">
          <div
            v-if="section.type === 'text'"
            class="article-view__text"
            v-html="section.content"
          />
          <QuizRenderer
            v-else-if="section.type === 'quiz'"
            :component="section.component"
            :quiz-props="section.props"
            @answer="markAnswered(i)"
          />
        </template>

        <div class="article-view__nav">
          <RouterLink v-if="prevArticle" :to="`/article/${prevArticle.id}`" class="article-view__nav-btn">
            ← {{ prevArticle.title }}
          </RouterLink>
          <div class="article-view__nav-spacer" />
          <RouterLink v-if="nextArticle" :to="`/article/${nextArticle.id}`" class="article-view__nav-btn article-view__nav-btn--next">
            {{ nextArticle.title }} →
          </RouterLink>
          <RouterLink v-else to="/test" class="article-view__nav-btn article-view__nav-btn--test">
            Пройти финальный тест →
          </RouterLink>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useRoute } from 'vue-router'
import { articles, getArticleById } from '@/data/articles.js'
import QuizRenderer from '@/components/quiz/QuizRenderer.vue'

const route = useRoute()
const article = computed(() => getArticleById(route.params.id))
const quizCount = computed(() => article.value?.sections.filter(s => s.type === 'quiz').length ?? 0)

const currentIndex = computed(() => articles.findIndex(a => a.id === route.params.id))
const prevArticle = computed(() => currentIndex.value > 0 ? articles[currentIndex.value - 1] : null)
const nextArticle = computed(() => currentIndex.value < articles.length - 1 ? articles[currentIndex.value + 1] : null)

const answeredSections = ref(new Set())

watch(() => route.params.id, () => { answeredSections.value = new Set() })

function markAnswered(sectionIndex) {
  answeredSections.value = new Set([...answeredSections.value, sectionIndex])
}

const answeredCount = computed(() => answeredSections.value.size)
const progressPct = computed(() => quizCount.value ? (answeredCount.value / quizCount.value) * 100 : 0)
</script>

<style lang="scss" scoped>
.article-view {
  &__not-found {
    padding: $space-16 0;
    text-align: center;
  }

  &__hero {
    padding: $space-10 0 $space-8;
    background: radial-gradient(ellipse at 50% 0%, rgba($color-primary, 0.08) 0%, transparent 60%);
    border-bottom: 1px solid $color-border;
    margin-bottom: 0;
  }

  &__back {
    display: inline-block;
    font-size: $font-size-sm;
    color: $color-text-muted;
    margin-bottom: $space-4;
    text-decoration: none;
    transition: color $transition-fast;

    &:hover { color: $color-primary; }
  }

  &__tag {
    display: inline-block;
    font-size: $font-size-xs;
    font-weight: $font-weight-semibold;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: $color-primary;
    background: $color-primary-light;
    padding: 2px 8px;
    border-radius: $radius-full;
    margin-bottom: $space-4;
  }

  &__title {
    font-size: clamp($font-size-2xl, 4vw, $font-size-4xl);
    letter-spacing: -0.02em;
    margin-bottom: $space-4;
  }

  &__desc {
    font-size: $font-size-lg;
    color: $color-text-muted;
    max-width: 620px;
    line-height: $line-height-relaxed;
    margin-bottom: $space-5;
  }

  &__meta {
    display: flex;
    gap: $space-6;
    font-size: $font-size-sm;
    color: $color-text-faint;
  }

  &__progress-bar-wrap {
    position: sticky;
    top: 60px;
    z-index: 90;
    background: rgba($color-bg, 0.95);
    backdrop-filter: blur(8px);
    border-bottom: 1px solid $color-border;
    padding: $space-2 0;
  }

  &__progress-info {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: $font-size-xs;
    color: $color-text-faint;
    margin-bottom: $space-1;
  }

  &__progress-done {
    color: $color-success;
    font-weight: $font-weight-semibold;
  }

  &__progress-track {
    height: 3px;
    background: $color-bg-elevated;
    border-radius: $radius-full;
    overflow: hidden;
  }

  &__progress-fill {
    height: 100%;
    background: linear-gradient(90deg, $color-primary, $color-success);
    border-radius: $radius-full;
    transition: width $transition-slow;
  }

  &__body {
    max-width: 720px;
    padding-top: $space-10;
  }

  &__text {
    :deep(h2) {
      font-size: $font-size-2xl;
      margin: $space-10 0 $space-4;
      padding-top: $space-6;
      border-top: 1px solid $color-border;
      letter-spacing: -0.01em;

      &:first-child { margin-top: 0; border-top: none; padding-top: 0; }
    }

    :deep(p) {
      color: $color-text;
      line-height: $line-height-relaxed;
      margin-bottom: $space-4;
      font-size: $font-size-lg;
    }

    :deep(ul), :deep(ol) {
      padding-left: $space-6;
      margin-bottom: $space-4;

      li {
        font-size: $font-size-lg;
        line-height: $line-height-relaxed;
        margin-bottom: $space-2;
        color: $color-text;
      }
    }

    :deep(strong) { color: $color-text; font-weight: $font-weight-semibold; }
    :deep(em) { color: $color-primary-hover; font-style: italic; }
    :deep(code) {
      font-family: $font-family-mono;
      font-size: 0.88em;
      background: $color-bg-elevated;
      padding: 2px 6px;
      border-radius: $radius-sm;
      color: $color-primary-hover;
    }
  }

  &__nav {
    display: flex;
    align-items: center;
    gap: $space-4;
    padding: $space-8 0;
    border-top: 1px solid $color-border;
    margin-top: $space-10;
  }

  &__nav-spacer { flex: 1; }

  &__nav-btn {
    padding: $space-3 $space-5;
    background: $color-bg-card;
    border: 1px solid $color-border;
    border-radius: $radius-md;
    color: $color-text;
    text-decoration: none;
    font-size: $font-size-sm;
    font-weight: $font-weight-medium;
    transition: all $transition-fast;
    max-width: 240px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;

    &:hover { border-color: $color-primary; color: $color-primary; }

    &--test {
      background: $color-primary;
      border-color: $color-primary;
      color: white;

      &:hover { background: $color-primary-hover; color: white; }
    }
  }
}
</style>
