<template>
  <header class="app-header">
    <div class="app-header__inner container">
      <RouterLink to="/" class="app-header__logo">
        <span class="app-header__logo-icon">⬡</span>
        <span class="app-header__logo-text">Prompt<span class="app-header__logo-accent">Academy</span></span>
      </RouterLink>

      <nav class="app-header__nav">
        <RouterLink to="/" class="app-header__link">Статьи</RouterLink>
        <RouterLink to="/test" class="app-header__link app-header__link--cta">Пройти тест</RouterLink>
      </nav>
    </div>
  </header>
</template>

<style lang="scss" scoped>
.app-header {
  position: sticky;
  top: 0;
  z-index: 100;
  background: rgba($color-bg, 0.9);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid $color-border;

  &__inner {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 60px;
  }

  &__logo {
    display: flex;
    align-items: center;
    gap: $space-2;
    text-decoration: none;
    color: $color-text;

    &:hover { color: $color-text; }
  }

  &__logo-icon {
    font-size: $font-size-2xl;
    color: $color-primary;
  }

  &__logo-text {
    font-size: $font-size-lg;
    font-weight: $font-weight-bold;
    letter-spacing: -0.02em;
  }

  &__logo-accent { color: $color-primary; }

  &__nav {
    display: flex;
    align-items: center;
    gap: $space-6;
  }

  &__link {
    font-size: $font-size-sm;
    font-weight: $font-weight-medium;
    color: $color-text-muted;
    text-decoration: none;
    transition: color $transition-fast;

    &:hover,
    &.router-link-active { color: $color-text; }

    &--cta {
      background: $color-primary;
      color: white;
      padding: $space-2 $space-4;
      border-radius: $radius-md;
      font-weight: $font-weight-semibold;

      &:hover,
      &.router-link-active {
        background: $color-primary-hover;
        color: white;
      }
    }
  }
}
</style>
