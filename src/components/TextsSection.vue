<template>
  <div class="texts-section">
    <!-- GRID VIEW -->
    <template v-if="!currentReading">
      <header class="texts-header">
        <button class="btn-back" @click="$emit('back')">← Назад на главную</button>
        <h1>Доступные тексты</h1>
      </header>

      <div class="texts-grid">
        <div v-for="text in texts" :key="text.file" class="text-card-wrapper">
          <div class="text-card-inner">
            <h3 class="text-title">{{ text.title }}</h3>
            
            <div class="meta-row">
              <span class="meta-badge language-badge">{{ text.language }}</span>
              <span class="meta-badge words-badge">{{ text.words }} слов</span>
            </div>
            
            <button class="read-button" @click="readText(text)">Читать</button>
          </div>
        </div>
      </div>

      <footer class="footer">
        <p>&copy; 2026 Read&Learn.  Подготовлено специально для вас.</p>
      </footer>
    </template>

    <!-- READING VIEW -->
    <template v-else>
      <header class="reading-header">
        <button class="btn-close" @click="closeReading">← Назад к списку</button>
        <div class="reading-title-section">
          <h1>{{ currentReading.title }}</h1>
          <p class="reading-meta">{{ currentReading.language }} • {{ currentReading.words }} слов</p>
        </div>
      </header>

      <div class="reading-controls">
        <div class="controls-wrapper">
          <div class="translation-options">
            <span class="options-label">Вид перевода:</span>
            <div class="mode-picker">
              <button 
                :class="['mode-btn', { active: translationMode === 'word' }]" 
                @click="translationMode = 'word'"
              >
                <span class="mode-icon">📝</span>
                По словам
              </button>
              <button 
                :class="['mode-btn', { active: translationMode === 'sentence' }]" 
                @click="translationMode = 'sentence'"
              >
                <span class="mode-icon">📖</span>
                По предложениям
              </button>
            </div>
          </div>
          <button class="btn-translate-full" @click="prepareFullTranslation">
            {{ fullTranslation ? '← Показать оригинал' : '→ Перевести весь текст' }}
          </button>
        </div>
      </div>

      <div v-if="!fullTranslation" class="reading-container">
        <article class="text-content">
          <p v-for="(paragraph, idx) in paragraphs" :key="idx" class="paragraph">
            <template v-if="translationMode === 'word'">
              <span
                v-for="(token, wIdx) in paragraph.split(/(\s+)/)"
                :key="`w-${idx}-${wIdx}`"
              >
                <span
                  v-if="token.trim()"
                  class="word clickable"
                  @click="showTooltip($event, token, 'word')"
                >
                  {{ token }}
                </span>
                <span v-else>
                  {{ token }}
                </span>
              </span>
            </template>
            <template v-else>
              <span
                v-for="(sent, sIdx) in paragraph.match(/[^.!?]+[.!?]+/g) || [paragraph]"
                :key="`s-${idx}-${sIdx}`"
                class="sentence clickable"
                @click="showTooltip($event, sent.trim(), 'sentence')"
              >{{ sent }}</span>
            </template>
          </p>
        </article>
      </div>

      <div v-else class="full-translation">
        <h3>Полный перевод</h3>
        <pre>{{ fullTranslation }}</pre>
      </div>

      <footer class="reading-footer">
        <button class="btn-finish" @click="closeReading">← Вернуться к списку</button>
      </footer>

    <!-- SMALL TOOLTIP (First step) -->
    <div v-if="showTranslationTooltip" class="translation-tooltip" :style="tooltipPosition">
      <div class="tooltip-header">{{ translationModalData.type === 'word' ? 'Слово' : 'Предложение' }}</div>
      <div class="tooltip-translation">{{ translationModalData.translation }}</div>
      <button v-if="translationModalData.explanation" class="tooltip-expand-btn" @click="showDetailedGrammar">Подробнее</button>
      <button class="tooltip-close-btn" @click="showTranslationTooltip = false">✕</button>
    </div>

    <!-- DETAILED GRAMMAR MODAL (Second step) -->
    <div v-if="showGrammarModal" class="grammar-modal" @click.self="showGrammarModal = false">
      <div class="grammar-content">
        <button class="modal-close" @click="showGrammarModal = false">✕</button>
        
        <div class="translation-block">
          <h3>{{ translationModalData.original }}</h3>
          <p class="translation-text"><strong>{{ translationModalData.translation }}</strong></p>
        </div>
        
        <div class="grammar-block">
          <h3>Грамматическое объяснение</h3>
          <p>{{ translationModalData.explanation }}</p>
        </div>
        
        <button class="modal-btn-close" @click="showGrammarModal = false">Закрыть</button>
      </div>
    </div>
  </template>
</div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

const props = defineProps({
  texts: {
    type: Array,
    required: true
  },
  translations: {
    type: Object,
    default: () => ({})
  }
})

defineEmits(['back'])

const currentReading = ref(null)
const textContent = ref('')
const translationMode = ref('word') // 'word' or 'sentence'
const fullTranslation = ref('')
const showTranslationTooltip = ref(false)
const showGrammarModal = ref(false)
const translationModalData = ref({
  original: '',
  translation: '',
  type: 'word',
  explanation: ''
})
const grammarRules = ref({})
const sentenceTranslations = ref({})
const tooltipPosition = ref({ top: '0px', left: '0px' })

const paragraphs = computed(() => {
  if (!textContent.value) return []
  return textContent.value.split('\n').filter(p => p.trim())
})

const readText = async (text) => {
  try {
    const response = await fetch(`/texts/${text.file}`)
    const content = await response.text()
    textContent.value = content
    currentReading.value = text
    
    // Load grammar rules
    const grammarRes = await fetch('/data/grammar.json')
    grammarRules.value = await grammarRes.json()
    
    // Load sentence translations
    const sentenceRes = await fetch('/data/sentence-translations.json')
    sentenceTranslations.value = await sentenceRes.json()
  } catch (error) {
    console.error('Error loading text:', error)
    alert('Ошибка при загрузке текста')
  }
}

const closeReading = () => {
  currentReading.value = null
  textContent.value = ''
}

const getTranslation = (word) => {
  const cleaned = word.toLowerCase().replace(/[.,!?;:—–\-"'«»()]/g, '')
  return props.translations?.[cleaned] || ''
}

const translateSentence = (sentence) => {
  // First, try to find exact sentence match in JSON
  if (currentReading.value && sentenceTranslations.value[currentReading.value.file]) {
    const sentences = sentenceTranslations.value[currentReading.value.file]
    for (const sentenceObj of sentences) {
      // Dynamically get the first key (source language) and second key (target language)
      const keys = Object.keys(sentenceObj)
      const sourceLanguageKey = keys[0] // First key is the source language
      const targetLanguageKey = keys[1] // Second key is the target language (usually 'russian')
      
      const originalText = sentenceObj[sourceLanguageKey]
      const translatedText = sentenceObj[targetLanguageKey]
      
      // Normalize sentences for comparison (remove extra spaces, punctuation)
      const normalizedOriginal = originalText.toLowerCase().trim()
      const normalizedInput = sentence.toLowerCase().trim()
      
      if (normalizedOriginal === normalizedInput || normalizedOriginal.includes(normalizedInput) || normalizedInput.includes(normalizedOriginal)) {
        return translatedText
      }
    }
  }
  
  // Fallback: translate word by word if no sentence match found
  const words = sentence.split(/\s+/).filter(Boolean)
  return words.map(w => getTranslation(w)).filter(Boolean).join(', ')
}

const showTooltip = (event, target, mode) => {
  const translation = mode === 'word' ? getTranslation(target) : translateSentence(target)
  translationModalData.value = {
    original: target,
    translation: translation || 'Перевод не найден',
    type: mode,
    explanation: getGrammarExplanation(target, mode)
  }
  
  showTranslationTooltip.value = true
  showGrammarModal.value = false
  
  // Position tooltip after render with proper bounds checking
  nextTick(() => {
    const tooltipEl = document.querySelector('.translation-tooltip')
    const triggerEl = event.currentTarget
    
    if (tooltipEl && triggerEl) {
      const triggerRect = triggerEl.getBoundingClientRect()
      const tooltipRect = tooltipEl.getBoundingClientRect()
      
      let left = triggerRect.left
      let top = triggerRect.top - tooltipRect.height - 8
      
      // If tooltip would go above viewport, show below instead
      if (top < 0) {
        top = triggerRect.bottom + 8
      }
      
      // If tooltip would exceed right edge, adjust left position
      if (left + tooltipRect.width > window.innerWidth - 10) {
        left = window.innerWidth - tooltipRect.width - 10
      }
      
      // If tooltip would go beyond left edge, move right
      if (left < 10) {
        left = 10
      }
      
      tooltipPosition.value = {
        top: top + 'px',
        left: left + 'px'
      }
    }
  })
}

const showDetailedGrammar = () => {
  showTranslationTooltip.value = false
  showGrammarModal.value = true
}

const getGrammarExplanation = (text, mode) => {
  if (mode === 'word') {
    const cleaned = text.toLowerCase().replace(/[.,!?;:—–\-"'«»()]/g, '')
    
    for (const [category, rules] of Object.entries(grammarRules.value)) {
      if (typeof rules === 'object' && rules[cleaned]) {
        return rules[cleaned]
      }
    }
    
    return `Объяснение для "${text}" не добавлено.`
  }
  
  return `Предложение: "${text}". Проанализируйте структуру.`
}

const prepareFullTranslation = async () => {
  if (!currentReading.value) return
  
  // Toggle: show/hide translation
  if (fullTranslation.value) {
    fullTranslation.value = ''
  } else {
    try {
      // Load translation from file: text1.txt → text1.translated.txt
      const translationFileName = currentReading.value.file.replace('.txt', '.translated.txt')
      const response = await fetch(`/texts/${translationFileName}`)
      if (response.ok) {
        const translation = await response.text()
        fullTranslation.value = translation
      } else {
        console.warn(`Translation file not found: ${translationFileName}`)
        fullTranslation.value = 'Перевод недоступен для этого текста'
      }
    } catch (error) {
      console.error('Error loading translation:', error)
      fullTranslation.value = 'Ошибка при загрузке перевода'
    }
  }
}
</script>

<style scoped>
.texts-section {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 48px 20px;
}

.texts-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 50px;
  padding-bottom: 24px;
  border-bottom: 3px solid #2d9f5a;
}

.btn-back {
  background: #2d9f5a;
  color: white;
  border: none;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: 'Cormorant Garamond', Georgia, serif;
}

.btn-back:hover {
  background: #1f6b3f;
  transform: translateX(-4px);
}

.texts-header h1 {
  font-size: 2rem;
  color: #051705;
  margin: 0;
  flex: 1;
}

.texts-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 32px;
  margin-bottom: 60px;
}

.text-card-wrapper {
  perspective: 1000px;
}

.text-card-inner {
  background: #ffffff;
  border: 4px solid #2d9f5a;
  border-radius: 16px;
  padding: 36px;
  min-height: 260px;
  display: flex;
  flex-direction: column;
  gap: 24px;
  box-shadow: 0 4px 16px rgba(45, 159, 90, 0.2);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.text-card-wrapper:hover .text-card-inner {
  transform: translateY(-12px);
  box-shadow: 0 16px 40px rgba(45, 159, 90, 0.3);
  border-color: #1f6b3f;
}

.text-title {
  margin: 0;
  font-size: 1.8rem;
  color: #051705;
  font-weight: 800;
  line-height: 1.2;
}

.meta-row {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.meta-badge {
  font-size: 1.05rem;
  font-weight: 700;
  padding: 10px 18px;
  border-radius: 10px;
  color: white;
  display: inline-block;
}

.language-badge {
  background: #2d9f5a;
}

.words-badge {
  background: #5ac96e;
}

.read-button {
  background: linear-gradient(135deg, #5ac96e 0%, #2d9f5a 100%);
  color: white;
  border: none;
  padding: 16px 24px;
  border-radius: 12px;
  font-size: 1.2rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 8px 24px rgba(45, 159, 90, 0.3);
  font-family: 'Cormorant Garamond', Georgia, serif;
  margin-top: auto;
}

.read-button:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 36px rgba(45, 159, 90, 0.4);
}

.read-button:active {
  transform: translateY(-1px);
}

.footer {
  text-align: center;
  padding: 32px 0;
  border-top: 2px solid rgba(45, 159, 90, 0.2);
  color: #7cb87c;
  font-size: 0.95rem;
}

/* READING VIEW STYLES */
.reading-header {
  display: flex;
  align-items: flex-start;
  gap: 24px;
  margin-bottom: 48px;
  padding-bottom: 32px;
  border-bottom: 3px solid #2d9f5a;
}

.btn-close {
  background: #2d9f5a;
  color: white;
  border: none;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: 'Cormorant Garamond', Georgia, serif;
  white-space: nowrap;
  flex-shrink: 0;
}

.btn-close:hover {
  background: #1f6b3f;
  transform: translateX(-4px);
}

.reading-title-section h1 {
  margin: 0 0 8px 0;
  font-size: 2.4rem;
  color: #051705;
  font-weight: 800;
  line-height: 1.2;
}

.reading-meta {
  margin: 0;
  font-size: 1.2rem;
  color: #1f6b3f;
  font-weight: 600;
}

.reading-controls {
  max-width: 850px;
  margin: 0 auto 40px;
  padding: 20px;
}

.controls-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 24px;
  flex-wrap: wrap;
}

.translation-options {
  display: flex;
  align-items: center;
  gap: 16px;
  flex: 1;
  min-width: 280px;
}

.options-label {
  font-size: 0.95rem;
  font-weight: 700;
  color: #1f6b3f;
  white-space: nowrap;
  font-family: 'Cormorant Garamond', Georgia, serif;
}

.mode-picker {
  display: flex;
  gap: 8px;
  background: rgba(45, 159, 90, 0.08);
  padding: 4px;
  border-radius: 12px;
  border: 2px solid rgba(45, 159, 90, 0.2);
}

.mode-btn {
  background: transparent;
  border: none;
  padding: 10px 18px;
  border-radius: 8px;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.25s ease;
  color: #1f6b3f;
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: 'Cormorant Garamond', Georgia, serif;
  white-space: nowrap;
}

.mode-btn .mode-icon {
  font-size: 1.1rem;
}

.mode-btn:hover {
  background: rgba(45, 159, 90, 0.12);
  transform: translateY(-1px);
}

.mode-btn.active {
  background: linear-gradient(135deg, #2d9f5a, #5ac96e);
  color: white;
  box-shadow: 0 4px 12px rgba(45, 159, 90, 0.25);
  transform: scale(1.02);
}

.mode-btn.active:hover {
  background: linear-gradient(135deg, #1f6b3f, #2d9f5a);
  transform: scale(1.03);
  box-shadow: 0 6px 16px rgba(45, 159, 90, 0.35);
}

.btn-translate-full {
  background: linear-gradient(135deg, #5ac96e, #2d9f5a);
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 10px;
  font-size: 0.95rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: 'Cormorant Garamond', Georgia, serif;
  box-shadow: 0 4px 16px rgba(45, 159, 90, 0.2);
  white-space: nowrap;
}

.btn-translate-full:hover {
  background: linear-gradient(135deg, #2d9f5a, #1f6b3f);
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(45, 159, 90, 0.35);
}

.btn-translate-full:active {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(45, 159, 90, 0.25);
}

.reading-container {
  max-width: 850px;
  margin: 0 auto;
  padding: 48px 32px;
  background: linear-gradient(135deg, rgba(255,255,255,0.98), rgba(241,248,241,0.6));
  border-radius: 20px;
  border: 2px solid rgba(45, 159, 90, 0.15);
  box-shadow: 0 8px 32px rgba(45, 159, 90, 0.1);
  margin-bottom: 48px;
  overflow-x: hidden;
}

.text-content {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.6rem;
  line-height: 1.9;
  color: #051705;
  text-align: left;
  white-space: normal;
}

.paragraph {
  margin: 0 0 28px 0;
  text-indent: 1.5em;
}

.paragraph:first-child {
  text-indent: 0;
}

.word, .sentence {
  display: inline;
  cursor: pointer;
  transition: all 0.15s ease;
  padding: 0 2px;
  border-radius: 3px;
}

.word.clickable:hover, .sentence.clickable:hover {
  background: rgba(45, 159, 90, 0.15);
  color: #1f6b3f;
}

.translation-tooltip {
  position: fixed;
  background: #ffffff;
  border: 2px solid #2d9f5a;
  border-radius: 12px;
  padding: 12px 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.18);
max-width: 280px;
      border-radius: 12px;
      padding: 12px 16px;
      background: white;
      word-break: break-word;
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.18);
      z-index: 2000;
  animation: slideUp 0.2s ease-out;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.tooltip-header {
  font-weight: 700;
  color: #1f6b3f;
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin: 0;
}

.tooltip-translation {
  font-size: 1rem;
  color: #051705;
  margin: 0;
  font-weight: 600;
  line-height: 1.4;
}

.tooltip-expand-btn {
  background: #2d9f5a;
  color: #fff;
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 0.85rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s ease;
  width: fit-content;
}

.tooltip-expand-btn:hover {
  background: #1f6b3f;
  transform: translateY(-1px);
}

.tooltip-close-btn {
  background: none;
  border: none;
  color: #999;
  cursor: pointer;
  font-size: 1.2rem;
  padding: 0;
  position: absolute;
  top: 6px;
  right: 6px;
  transition: color 0.2s;
}

.tooltip-close-btn:hover {
  color: #1f6b3f;
}

.grammar-modal {
  position: fixed;
  inset: 0;
  background: rgba(5, 20, 5, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 3000;
  animation: fadeIn 0.2s ease-out;
}

.grammar-content {
  max-width: 620px;
  background: #ffffff;
  border-radius: 16px;
  padding: 28px;
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.2);
  position: relative;
}

@keyframes slideUp {
  from { 
    opacity: 0;
    transform: translateY(8px);
  }
  to { 
    opacity: 1;
    transform: translateY(0);
  }
}

.modal-close {
  position: absolute;
  top: 12px;
  right: 12px;
  background: none;
  border: none;
  font-size: 1.4rem;
  color: #999;
  cursor: pointer;
  transition: color 0.2s;
}

.modal-close:hover {
  color: #1f6b3f;
}

.translation-block {
  margin-bottom: 24px;
  padding-bottom: 20px;
  border-bottom: 2px solid #e6f7e8;
}

.translation-block h3 {
  margin: 0 0 12px 0;
  font-size: 1.1rem;
  color: #1f6b3f;
  font-weight: 700;
}

.translation-text {
  margin: 0;
  font-size: 1.3rem;
  color: #051705;
  line-height: 1.6;
}

.full-translation {
  max-width: 850px;
  margin: 0 auto 48px;
  padding: 48px 32px;
  background: linear-gradient(135deg, rgba(255,255,255,0.98), rgba(241,248,241,0.6));
  border-radius: 20px;
  border: 2px solid rgba(45, 159, 90, 0.15);
  box-shadow: 0 8px 32px rgba(45, 159, 90, 0.1);
}

.full-translation h3 {
  margin: 0 0 24px 0;
  font-size: 1.3rem;
  color: #1f6b3f;
  font-weight: 700;
  text-align: center;
}

.full-translation pre {
  font-family: 'Cormorant Garamond', Georgia, serif;
  font-size: 1.6rem;
  line-height: 1.9;
  color: #051705;
  white-space: pre-wrap;
  text-align: left;
  margin: 0;
  text-indent: 1.5em;
}

.reading-footer {
  max-width: 850px;
  margin: 48px auto 0;
  padding: 24px 0;
  text-align: center;
  border-top: 2px solid rgba(45, 159, 90, 0.15);
}

.btn-finish {
  background: linear-gradient(135deg, #2d9f5a, #5ac96e);
  color: white;
  border: none;
  padding: 14px 28px;
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  font-family: 'Cormorant Garamond', Georgia, serif;
  box-shadow: 0 4px 16px rgba(45, 159, 90, 0.2);
}

.btn-finish:hover {
  background: linear-gradient(135deg, #1f6b3f, #2d9f5a);
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(45, 159, 90, 0.35);
}

.btn-finish:active {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(45, 159, 90, 0.25);
}

.grammar-block {
  margin-bottom: 24px;
}

.grammar-block h3 {
  margin: 0 0 12px 0;
  font-size: 1.1rem;
  color: #1f6b3f;
  font-weight: 700;
}

.grammar-block p {
  margin: 0;
  color: #1f4620;
  font-size: 0.95rem;
  line-height: 1.6;
}

.modal-btn-close {
  width: 100%;
  background: linear-gradient(135deg, #2d9f5a, #5ac96e);
  color: #ffffff;
  border: none;
  padding: 12px 20px;
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.2s ease;
}

.modal-btn-close:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(45, 159, 90, 0.3);
}

@media (max-width: 768px) {
  .texts-section {
    padding: 32px 16px;
  }

  .texts-grid {
    grid-template-columns: 1fr;
    gap: 24px;
  }

  .texts-header {
    flex-direction: column;
    align-items: flex-start;
    margin-bottom: 32px;
  }

  .reading-container {
    padding: 24px 16px;
    margin-bottom: 24px;
  }

  .text-content {
    font-size: 1.4rem;
    line-height: 1.7;
  }

  .paragraph {
    margin: 0 0 20px 0;
    text-indent: 1em;
  }

  .reading-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .reading-title-section h1 {
    font-size: 1.8rem;
  }

  .texts-header h1 {
    font-size: 1.6rem;
  }

  .text-card-inner {
    padding: 28px;
    min-height: auto;
  }

  .text-title {
    font-size: 1.4rem;
  }

  .read-button {
    font-size: 1rem;
    padding: 14px 20px;
  }
}
</style>