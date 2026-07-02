<template>
  <div class="home">
    <section class="home__hero">
      <div class="container">
        <div class="home__hero-badge">Образовательный портал</div>
        <h1 class="home__hero-title">
          Prompt Engineering<br>
          <span class="home__hero-accent">Academy</span>
        </h1>
        <p class="home__hero-desc">
          Научитесь писать эффективные запросы к AI — не через списки советов,
          а через практику прямо в тексте статей.
        </p>
        <div class="home__hero-actions">
          <RouterLink to="/article/how-llm-works" class="home__btn home__btn--primary">
            Начать обучение
          </RouterLink>
          <RouterLink to="/test" class="home__btn home__btn--outline">
            Пройти финальный тест
          </RouterLink>
        </div>
        <div class="home__hero-stats">
          <div class="home__stat">
            <span class="home__stat-val">{{ articles.length }}</span>
            <span class="home__stat-label">Статей</span>
          </div>
          <div class="home__stat">
            <span class="home__stat-val">8</span>
            <span class="home__stat-label">Типов упражнений</span>
          </div>
          <div class="home__stat">
            <span class="home__stat-val">{{ fullTest.questions.length }}</span>
            <span class="home__stat-label">Вопросов в тесте</span>
          </div>
        </div>
      </div>
    </section>

    <section class="home__articles container">
      <h2 class="home__section-title">Статьи</h2>
      <div class="home__articles-grid">
        <RouterLink
          v-for="article in articles"
          :key="article.id"
          :to="`/article/${article.id}`"
          class="home__article-card"
        >
          <span class="home__article-tag">{{ article.tag }}</span>
          <h3 class="home__article-title">{{ article.title }}</h3>
          <p class="home__article-desc">{{ article.description }}</p>
          <div class="home__article-meta">
            <span>{{ article.readTime }} мин чтения</span>
            <span class="home__article-quiz-count">
              {{ quizCount(article) }} упражнений
            </span>
          </div>
        </RouterLink>
      </div>
    </section>

    <section class="home__components container">
      <h2 class="home__section-title">Типы упражнений</h2>
      <div class="home__components-grid">
        <div v-for="comp in componentTypes" :key="comp.name" class="home__comp-card">
          <span class="home__comp-icon">{{ comp.icon }}</span>
          <h4 class="home__comp-name">{{ comp.name }}</h4>
          <p class="home__comp-desc">{{ comp.desc }}</p>
        </div>
      </div>
    </section>

    <section class="home__cta container">
      <div class="home__cta-box">
        <h2>Готовы проверить знания?</h2>
        <p>12 вопросов, 7 типов заданий, финальный результат с рекомендациями.</p>
        <RouterLink to="/test" class="home__btn home__btn--primary">Начать финальный тест</RouterLink>
      </div>
    </section>
  </div>
</template>

<script setup>
import { articles } from '@/data/articles.js'
import { fullTest } from '@/data/test.js'

function quizCount(article) {
  return article.sections.filter(s => s.type === 'quiz').length
}

const componentTypes = [
  { icon: '○', name: 'Один/несколько ответов', desc: 'Классический формат с поддержкой частично правильных ответов' },
  { icon: '↔', name: 'Сопоставь пары', desc: 'Drag-and-drop или клики: свяжи термин с определением' },
  { icon: '_', name: 'Заполни пропуск', desc: 'Введи нужное слово в нужном месте текста' },
  { icon: '✓', name: 'Верно/Неверно', desc: 'Оцени утверждение с детальным объяснением' },
  { icon: '⠿', name: 'Расставь по порядку', desc: 'Drag-and-drop для правильной последовательности шагов' },
  { icon: '⚖', name: 'Сравни промпты', desc: 'Кастомный: выбери более эффективный запрос к AI' },
  { icon: '⬡', name: 'Собери промпт', desc: 'Drag-and-drop сборка промпта из правильных блоков' },
  { icon: '🔍', name: 'Найди галлюцинацию', desc: 'Кликни на ложные фрагменты в ответе AI' }
]
</script>

<style lang="scss" scoped>
.home {
  &__hero {
    padding: $space-20 0 $space-16;
    background: radial-gradient(ellipse at 50% 0%, rgba($color-primary, 0.12) 0%, transparent 70%);
    border-bottom: 1px solid $color-border;
    margin-bottom: $space-16;
    text-align: center;
  }

  &__hero-badge {
    display: inline-block;
    font-size: $font-size-xs;
    font-weight: $font-weight-semibold;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    color: $color-primary;
    background: $color-primary-light;
    padding: $space-1 $space-4;
    border-radius: $radius-full;
    margin-bottom: $space-5;
    border: 1px solid rgba($color-primary, 0.3);
  }

  &__hero-title {
    font-size: clamp(2.5rem, 6vw, 4rem);
    font-weight: $font-weight-bold;
    line-height: 1.1;
    margin-bottom: $space-5;
    letter-spacing: -0.03em;
  }

  &__hero-accent {
    background: linear-gradient(135deg, $color-primary, #c084fc);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  &__hero-desc {
    font-size: $font-size-lg;
    color: $color-text-muted;
    max-width: 560px;
    margin: 0 auto $space-8;
    line-height: $line-height-relaxed;
  }

  &__hero-actions {
    display: flex;
    gap: $space-4;
    justify-content: center;
    flex-wrap: wrap;
    margin-bottom: $space-10;
  }

  &__hero-stats {
    display: flex;
    gap: $space-8;
    justify-content: center;
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
    color: $color-primary;
  }

  &__stat-label {
    font-size: $font-size-xs;
    color: $color-text-faint;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  &__btn {
    display: inline-flex;
    align-items: center;
    gap: $space-2;
    padding: $space-3 $space-6;
    border-radius: $radius-md;
    font-size: $font-size-base;
    font-weight: $font-weight-semibold;
    text-decoration: none;
    transition: all $transition-fast;

    &--primary {
      background: $color-primary;
      color: white;

      &:hover { background: $color-primary-hover; color: white; }
    }

    &--outline {
      background: transparent;
      border: 2px solid $color-border;
      color: $color-text;

      &:hover { border-color: $color-primary; color: $color-primary; }
    }
  }

  &__section-title {
    font-size: $font-size-2xl;
    margin-bottom: $space-6;
    color: $color-text;
  }

  &__articles {
    margin-bottom: $space-16;
  }

  &__articles-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
    gap: $space-5;
  }

  &__article-card {
    background: $color-bg-card;
    border: 1px solid $color-border;
    border-radius: $radius-lg;
    padding: $space-5;
    text-decoration: none;
    color: $color-text;
    transition: all $transition-base;
    display: flex;
    flex-direction: column;
    gap: $space-3;

    &:hover {
      border-color: $color-primary;
      transform: translateY(-2px);
      box-shadow: $shadow-md;
      color: $color-text;
    }
  }

  &__article-tag {
    font-size: $font-size-xs;
    font-weight: $font-weight-semibold;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: $color-primary;
    background: $color-primary-light;
    padding: 2px 8px;
    border-radius: $radius-full;
    width: fit-content;
  }

  &__article-title {
    font-size: $font-size-lg;
    font-weight: $font-weight-bold;
    line-height: $line-height-tight;
  }

  &__article-desc {
    font-size: $font-size-sm;
    color: $color-text-muted;
    line-height: $line-height-relaxed;
    flex: 1;
    margin: 0;
  }

  &__article-meta {
    display: flex;
    justify-content: space-between;
    font-size: $font-size-xs;
    color: $color-text-faint;
    padding-top: $space-2;
    border-top: 1px solid $color-border;
  }

  &__article-quiz-count { color: $color-primary; }

  &__components {
    margin-bottom: $space-16;
  }

  &__components-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: $space-4;
  }

  &__comp-card {
    background: $color-bg-card;
    border: 1px solid $color-border;
    border-radius: $radius-lg;
    padding: $space-4;
    display: flex;
    flex-direction: column;
    gap: $space-2;
  }

  &__comp-icon {
    font-size: $font-size-xl;
    color: $color-primary;
    font-family: $font-family-mono;
  }

  &__comp-name {
    font-size: $font-size-sm;
    font-weight: $font-weight-semibold;
  }

  &__comp-desc {
    font-size: $font-size-xs;
    color: $color-text-muted;
    line-height: $line-height-relaxed;
    margin: 0;
  }

  &__cta {
    margin-bottom: $space-8;
  }

  &__cta-box {
    background: linear-gradient(135deg, rgba($color-primary, 0.1), rgba(#c084fc, 0.1));
    border: 1px solid rgba($color-primary, 0.3);
    border-radius: $radius-xl;
    padding: $space-10;
    text-align: center;

    h2 {
      font-size: $font-size-2xl;
      margin-bottom: $space-3;
    }

    p {
      color: $color-text-muted;
      margin-bottom: $space-6;
      font-size: $font-size-lg;
    }
  }
}
</style>
