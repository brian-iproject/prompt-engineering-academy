<template>
  <div class="test-view">
    <div class="test-view__hero">
      <div class="container">
        <RouterLink to="/" class="test-view__back">← На главную</RouterLink>
        <h1 class="test-view__title">{{ fullTest.title }}</h1>
        <p class="test-view__desc">{{ fullTest.description }}</p>
      </div>
    </div>

    <div class="test-view__body container">
      <QuizRunner :questions="fullTest.questions" />
    </div>
  </div>
</template>

<script setup>
import { fullTest } from '@/data/test.js'
import QuizRunner from '@/components/QuizRunner.vue'
</script>

<style lang="scss" scoped>
.test-view {
  &__hero {
    padding: $space-8 0 $space-6;
    border-bottom: 1px solid $color-border;
    margin-bottom: $space-8;
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

  &__title {
    font-size: $font-size-3xl;
    letter-spacing: -0.02em;
    margin-bottom: $space-3;
  }

  &__desc {
    font-size: $font-size-lg;
    color: $color-text-muted;
    margin: 0;
  }

  &__body {
    max-width: 720px;
  }
}
</style>
