<script setup>
import { ref, onMounted } from 'vue'
import TextsSection from './components/TextsSection.vue'

const currentSection = ref('home')
const texts = ref([])
const translations = ref({})

onMounted(async () => {
  try {
    const csvResponse = await fetch('/texts/texts.csv')
    const csvText = await csvResponse.text()
    const lines = csvText.trim().split('\n').slice(1)
    
    texts.value = lines.map(line => {
      const [file, title, language, words] = line.split(',')
      return { file, title, language, words: parseInt(words) }
    })
  } catch (e) {
    console.error('Error loading texts:', e)
  }
  
  try {
    const transResponse = await fetch('/data/translations.json')
    translations.value = await transResponse.json()
  } catch (e) {
    console.error('Error loading translations:', e)
  }
})

const navigateToTexts = () => {
  currentSection.value = 'texts'
}

const navigateHome = () => {
  currentSection.value = 'home'
}
</script>

<template>
  <div class="welcome-page">
    <transition name="fade">
      <!-- Home Section -->
      <div v-if="currentSection === 'home'" class="container" key="home">
        <header class="header">
          <div class="brand">
            <div class="logo" aria-hidden="true"></div>
            <div class="titles">
              <h1>Читай в оригинале</h1>
              <p class="subtitle">Интерактивное чтение с переводом слов прямо во время чтения</p>
            </div>
          </div>
        </header>

        <main class="hero-section">
          <div class="hero-content">
            <h2>Учись, читая</h2>
            <p>Наведи курсор на любое слово — узнаешь его перевод мгновенно. На мобильном просто коснись слова.</p>
            <div class="actions">
              <button class="btn primary" @click="navigateToTexts">Начать чтение</button>
            </div>
          </div>
        </main>

        <section class="features">
          <div class="feature-card">
            <div class="feature-dot"></div>
            <h3>Интерактивно</h3>
            <p>Наводишь на слово — видишь перевод мгновенно</p>
          </div>
          <div class="feature-card">
            <div class="feature-dot"></div>
            <h3>На любом устройстве</h3>
            <p>Работает одинаково хорошо на ПК, планшете и смартфоне</p>
          </div>
          <div class="feature-card">
            <div class="feature-dot"></div>
            <h3>Учись свободно</h3>
            <p>Читай интересные тексты и развивай словарный запас естественным путём</p>
          </div>
        </section>

        <footer class="footer">
          <p>&copy; 2026 Read&Learn.  Подготовлено специально для вас.</p>
        </footer>
      </div>

      <!-- Texts Section -->
      <TextsSection 
        v-else-if="currentSection === 'texts'"
        :texts="texts" 
        :translations="translations"
        @back="navigateHome"
        key="texts"
      />
    </transition>
  </div>
</template>

<style scoped>
:root {
  --bg: #ffffff;
  --green-light: #f1f8f1;
  --green-accent: #e6f7e8;
  --green-400: #5ac96e;
  --green-500: #2d9f5a;
  --green-600: #1f6b3f;
  --text: #0d2b0d;
  --text-light: #1f4620;
}

.welcome-page {
  font-family: 'Cormorant Garamond', Georgia, serif;
  color: var(--text-light);
  background: none;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 48px 20px;
}

.container {
  width: 100%;
  max-width: 1100px;
  display: flex;
  flex-direction: column;
  gap: 40px;
}

.header {
  display: flex;
  align-items: center;
  text-align: center;
  justify-content: center;
}

.brand {
  display: flex;
  gap: 16px;
  align-items: center;
  flex-direction: column;
  width: 100%;
}

.logo {
  width: 64px;
  height: 64px;
  border-radius: 16px;
  background: linear-gradient(135deg, #2d9f5a, #1f6b3f);
  box-shadow: 0 8px 24px rgba(45, 159, 90, 0.3);
}

.titles {
  text-align: center;
}

.titles h1 {
  margin: 0;
  font-weight: 800;
  font-size: 2.6rem;
  color: #051705;
  letter-spacing: -0.5px;
}

.subtitle {
  margin: 8px 0 0;
  color: #0d4620;
  font-size: 1.3rem;
  font-weight: 500;
}

.hero-section {
  background: linear-gradient(135deg, rgba(255,255,255,0.98), rgba(241,248,241,0.8));
  border-radius: 20px;
  padding: 48px;
  text-align: center;
  border: 2px solid rgba(45, 159, 90, 0.2);
  box-shadow: 0 12px 40px rgba(45, 159, 90, 0.12);
}

.hero-content h2 {
  margin: 0 0 16px 0;
  font-size: 2.2rem;
  color: #1f6b3f;
  font-weight: 700;
}

.hero-content p {
  margin: 0 0 28px 0;
  color: #1f4620;
  font-size: 1.25rem;
  line-height: 1.6;
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}

.actions {
  display: flex;
  gap: 12px;
  justify-content: center;
}

.btn {
  font-weight: 600;
  padding: 12px 24px;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  font-size: 0.95rem;
  transition: all 0.3s ease;
}

.btn.primary {
  background: linear-gradient(135deg, #5ac96e, #2d9f5a);
  color: #ffffff;
  box-shadow: 0 8px 24px rgba(45, 159, 90, 0.35);
}

.btn.primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 32px rgba(45, 159, 90, 0.45);
}

.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}

.feature-card {
  background: linear-gradient(135deg, rgba(255,255,255,0.98), rgba(241,248,241,0.7));
  border-radius: 16px;
  padding: 32px;
  text-align: center;
  border: 2px solid rgba(45, 159, 90, 0.18);
  box-shadow: 0 6px 20px rgba(45, 159, 90, 0.1);
  transition: all 0.3s ease;
}

.feature-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 32px rgba(45, 159, 90, 0.18);
  border-color: rgba(45, 159, 90, 0.3);
  background: linear-gradient(135deg, rgba(255,255,255,0.99), rgba(230, 247, 232, 0.9));
}

.feature-dot {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: linear-gradient(135deg, #5ac96e, #2d9f5a);
  margin: 0 auto 16px;
  box-shadow: 0 6px 16px rgba(45, 159, 90, 0.25);
}

.feature-card h3 {
  margin: 0 0 8px 0;
  font-size: 1.4rem;
  color: #1f6b3f;
  font-weight: 600;
}

.feature-card p {
  margin: 0;
  color: #1f4620;
  font-size: 1.05rem;
  line-height: 1.5;
}

.footer {
  text-align: center;
  padding: 20px 0;
  color: #7cb87c;
  font-size: 0.9rem;
  border-top: 2px solid rgba(45, 159, 90, 0.15);
  margin-top: 20px;
}

/* Responsive */
@media (max-width: 900px) {
  .hero-section {
    padding: 32px;
  }

  .titles h1 {
    font-size: 1.6rem;
  }

  .hero-content h2 {
    font-size: 1.4rem;
  }
}

@media (max-width: 640px) {
  .welcome-page {
    padding: 28px 16px;
  }

  .container {
    gap: 28px;
  }

  .header {
    margin-bottom: 16px;
  }

  .brand {
    gap: 12px;
  }

  .logo {
    width: 52px;
    height: 52px;
  }

  .titles h1 {
    font-size: 1.4rem;
  }

  .subtitle {
    font-size: 0.9rem;
  }

  .hero-section {
    padding: 24px;
  }

  .hero-content h2 {
    font-size: 1.2rem;
  }

  .hero-content p {
    font-size: 0.95rem;
  }

  .feature-card {
    padding: 24px;
  }
}

/* Animations */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
</style>