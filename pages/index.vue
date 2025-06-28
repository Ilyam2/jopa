<template>
  <div class="clicker-container" :class="{ 'light-theme': !darkTheme }">
    <!-- Анимированные частицы фона -->
    <div class="particles">
      <div v-for="i in 50" :key="i" class="particle" :style="{ left: Math.random() * 100 + '%', animationDelay: Math.random() * 5 + 's' }"></div>
    </div>
    
    <!-- Панель управления -->
    <div class="control-panel">
      <button @click="toggleSound" class="control-btn" :class="{ 'active': soundEnabled }">
        <span class="control-icon">{{ soundEnabled ? '🔊' : '🔇' }}</span>
      </button>
      <button @click="toggleTheme" class="control-btn" :class="{ 'active': darkTheme }">
        <span class="control-icon">{{ darkTheme ? '🌙' : '☀️' }}</span>
      </button>
    </div>
    
    <!-- Основной контент игры -->
    <div class="game-content">
      <div class="score-section">
        <h1 class="game-title">🚀 Космический Кликер</h1>
        <div class="score-display">
          <span class="score-label">Счёт:</span>
          <span class="score-value">{{ formatNumber(score) }}</span>
        </div>
        <div class="stats-row">
          <div class="multiplier">
            <span class="multiplier-label">Множитель:</span>
            <span class="multiplier-value">x{{ multiplier }}</span>
          </div>
          <div class="cps-display">
            <span class="cps-label">Кликов/сек:</span>
            <span class="cps-value">{{ clicksPerSecond }}</span>
          </div>
        </div>
        <div class="total-clicks">
          <span class="clicks-label">Всего кликов:</span>
          <span class="clicks-value">{{ formatNumber(totalClicks) }}</span>
        </div>
      </div>

      <!-- Главная кнопка клика -->
      <div class="click-section">
        <button 
          @click="click()" 
          class="click-button"
          :class="{ 'clicked': isClicked }"
          @mousedown="isClicked = true"
          @mouseup="isClicked = false"
          @mouseleave="isClicked = false"
        >
          <div class="button-content">
            <div class="button-icon">⭐</div>
            <div class="button-text">КЛИК!</div>
          </div>
          <div class="button-glow"></div>
        </button>
      </div>

      <!-- Секция достижений -->
      <div class="achievements-section">
        <h2 class="achievements-title">🏆 Достижения</h2>
        <div class="achievements-grid">
          <div 
            v-for="achievement in achievements" 
            :key="achievement.id"
            class="achievement-card"
            :class="{ 'unlocked': achievement.unlocked }"
          >
            <div class="achievement-icon">{{ achievement.icon }}</div>
            <div class="achievement-info">
              <div class="achievement-name">{{ achievement.name }}</div>
              <div class="achievement-description">{{ achievement.description }}</div>
            </div>
            <div class="achievement-status">
              <span v-if="achievement.unlocked" class="status-unlocked">✓</span>
              <span v-else class="status-locked">🔒</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Секция улучшений -->
      <div class="upgrades-section">
        <h2 class="upgrades-title">🚀 Улучшения</h2>
        <div class="upgrades-grid">
          <div 
            v-for="upgrade in upgrades" 
            :key="upgrade.id"
            class="upgrade-card"
            :class="{ 'affordable': score >= upgrade.cost }"
            @click="buyUpgrade(upgrade)"
          >
            <div class="upgrade-icon">{{ upgrade.icon }}</div>
            <div class="upgrade-info">
              <div class="upgrade-name">{{ upgrade.name }}</div>
              <div class="upgrade-cost">{{ formatNumber(upgrade.cost) }}</div>
            </div>
            <div class="upgrade-count">{{ upgrade.count }}</div>
          </div>
        </div>
      </div>

      <!-- Летающие частицы при клике -->
      <div v-for="particle in clickParticles" :key="particle.id" class="click-particle" :style="particle.style">
        {{ particle.text || `+${particle.value}` }}
      </div>

      <!-- Уведомления о достижениях -->
      <div v-for="notification in achievementNotifications" :key="notification.id" class="achievement-notification" :style="notification.style">
        <div class="notification-content">
          <div class="notification-icon">🏆</div>
          <div class="notification-text">
            <div class="notification-title">Достижение разблокировано!</div>
            <div class="notification-achievement">{{ notification.achievement.name }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue'

// ===== РЕАКТИВНЫЕ ПЕРЕМЕННЫЕ =====
// Основной счёт игрока
const score = ref<number>(0)
// Множитель кликов
const multiplier = ref<number>(1)
// Состояние нажатия кнопки (для анимации)
const isClicked = ref<boolean>(false)
// Массив частиц для визуальных эффектов
const clickParticles = ref<any[]>([])
// Уникальный ID для частиц
let particleId = 0
// Звуковые эффекты включены
const soundEnabled = ref<boolean>(true)
// Тёмная тема включена
const darkTheme = ref<boolean>(true)
// Массив достижений
const achievements = ref([
  { id: 1, name: 'Первый клик!', icon: '🎯', unlocked: true, description: 'Сделайте первый клик' },
  { id: 2, name: 'Кликер', icon: '👆', unlocked: false, description: '100 кликов' },
  { id: 3, name: 'Мастер кликов', icon: '👑', unlocked: false, description: '1,000 кликов' },
  { id: 4, name: 'Легенда', icon: '⭐', unlocked: false, description: '10,000 кликов' },
  { id: 5, name: 'Богач', icon: '💰', unlocked: false, description: '1,000,000 очков' },
  { id: 6, name: 'Миллиардер', icon: '💎', unlocked: false, description: '1,000,000,000 очков' }
])
// Уведомления о достижениях
const achievementNotifications = ref<any[]>([])
// Общее количество кликов
const totalClicks = ref<number>(0)
// Время последнего клика для подсчёта CPS
const lastClickTime = ref<number>(0)
// Кликов в секунду
const clicksPerSecond = ref<number>(0)

// ===== КОНФИГУРАЦИЯ УЛУЧШЕНИЙ =====
// Массив всех доступных улучшений
const upgrades = ref([
  { id: 1, name: 'Авто-кликер', icon: '🤖', cost: 1, count: 0, effect: 'autoClick' },
  { id: 2, name: 'Множитель', icon: '⚡', cost: 1, count: 0, effect: 'multiplier' },
  { id: 3, name: 'Мега-клик', icon: '💎', cost: 1, count: 0, effect: 'megaClick' },
  { id: 4, name: 'Искривление времени', icon: '⏰', cost: 1, count: 0, effect: 'timeWarp' },
  { id: 5, name: 'Квантовый буст', icon: '🌌', cost: 1, count: 0, effect: 'quantumBoost' },
  { id: 6, name: 'Камень бесконечности', icon: '💎', cost: 1, count: 0, effect: 'infinityStone' }
])

// ===== ПЕРЕМЕННЫЕ ДЛЯ АВТО-КЛИКЕРА =====
let autoClickInterval: NodeJS.Timeout | null = null

// ===== ОБРАБОТЧИКИ СОБЫТИЙ =====
/**
 * Обработчик нажатия клавиш для сброса прогресса
 * @param event - событие нажатия клавиши
 */
const handleKeyDown = (event: KeyboardEvent) => {
  if (event.ctrlKey && event.key === 'r') {
    event.preventDefault()
    if (confirm('Сбросить весь прогресс игры?')) {
      resetGameProgress()
    }
  }
}

// ===== ФУНКЦИИ УТИЛИТЫ =====

/**
 * Форматирует большие числа в читаемый вид (1K, 1M, 1B, 1T)
 * @param num - число для форматирования
 * @returns отформатированная строка
 */
const formatNumber = (num: number): string => {
  if (num >= 1e12) return (num / 1e12).toFixed(1) + 'T'
  if (num >= 1e9) return (num / 1e9).toFixed(1) + 'B'
  if (num >= 1e6) return (num / 1e6).toFixed(1) + 'M'
  if (num >= 1e3) return (num / 1e3).toFixed(1) + 'K'
  return num.toString()
}

/**
 * Создаёт визуальную частицу при клике
 * @param x - X координата клика
 * @param y - Y координата клика
 * @param value - значение для отображения
 */
const createClickParticle = (x: number, y: number, value: number) => {
  const particle = {
    id: particleId++,
    value: value,
    style: {
      left: x + 'px',
      top: y + 'px',
      animationDelay: '0s'
    }
  }
  clickParticles.value.push(particle)
  
  // Удаляем частицу через 1 секунду
  setTimeout(() => {
    clickParticles.value = clickParticles.value.filter(p => p.id !== particle.id)
  }, 1000)
}

// ===== ОСНОВНЫЕ ФУНКЦИИ ИГРЫ =====

/**
 * Обработчик клика по главной кнопке
 * @param event - событие мыши (опционально)
 */
const click = (event?: MouseEvent) => {
  const clickValue = multiplier.value
  score.value += clickValue
  totalClicks.value++
  
  // Обновляем CPS
  const now = Date.now()
  if (lastClickTime.value > 0) {
    const timeDiff = now - lastClickTime.value
    if (timeDiff < 1000) {
      clicksPerSecond.value = Math.min(clicksPerSecond.value + 1, 100)
    } else {
      clicksPerSecond.value = 1
    }
  }
  lastClickTime.value = now
  
  // Создаём визуальный эффект частицы
  if (event) {
    createClickParticle(event.clientX, event.clientY, clickValue)
  }
  
  // Воспроизводим звук клика
  playClickSound()
  
  // Проверяем достижения
  checkAchievements()
}

/**
 * Воспроизводит звук клика
 */
const playClickSound = () => {
  if (!soundEnabled.value) return
  
  try {
    // Создаём простой звук клика
    const audioContext = new (window.AudioContext || (window as any).webkitAudioContext)()
    const oscillator = audioContext.createOscillator()
    const gainNode = audioContext.createGain()
    
    oscillator.connect(gainNode)
    gainNode.connect(audioContext.destination)
    
    oscillator.frequency.setValueAtTime(800, audioContext.currentTime)
    oscillator.frequency.exponentialRampToValueAtTime(400, audioContext.currentTime + 0.1)
    
    gainNode.gain.setValueAtTime(0.1, audioContext.currentTime)
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1)
    
    oscillator.start(audioContext.currentTime)
    oscillator.stop(audioContext.currentTime + 0.1)
  } catch (error) {
    console.log('Звук недоступен:', error)
  }
}

/**
 * Проверяет и разблокирует достижения
 */
const checkAchievements = () => {
  achievements.value.forEach(achievement => {
    if (achievement.unlocked) return
    
    let shouldUnlock = false
    
    switch (achievement.id) {
      case 2: // Кликер
        shouldUnlock = totalClicks.value >= 100
        break
      case 3: // Мастер кликов
        shouldUnlock = totalClicks.value >= 1000
        break
      case 4: // Легенда
        shouldUnlock = totalClicks.value >= 10000
        break
      case 5: // Богач
        shouldUnlock = score.value >= 1000000
        break
      case 6: // Миллиардер
        shouldUnlock = score.value >= 1000000000
        break
    }
    
    if (shouldUnlock) {
      achievement.unlocked = true
      showAchievementNotification(achievement)
    }
  })
}

/**
 * Показывает уведомление о достижении
 * @param achievement - достижение для показа
 */
const showAchievementNotification = (achievement: any) => {
  const notification = {
    id: Date.now(),
    achievement: achievement,
    style: {
      animationDelay: '0s'
    }
  }
  
  achievementNotifications.value.push(notification)
  
  // Удаляем уведомление через 3 секунды
  setTimeout(() => {
    achievementNotifications.value = achievementNotifications.value.filter(n => n.id !== notification.id)
  }, 3000)
  
  // Воспроизводим звук достижения
  if (soundEnabled.value) {
    try {
      const audioContext = new (window.AudioContext || (window as any).webkitAudioContext)()
      const oscillator = audioContext.createOscillator()
      const gainNode = audioContext.createGain()
      
      oscillator.connect(gainNode)
      gainNode.connect(audioContext.destination)
      
      // Создаём мелодию достижения
      const frequencies = [523, 659, 784, 1047] // C, E, G, C
      frequencies.forEach((freq, index) => {
        const osc = audioContext.createOscillator()
        const gain = audioContext.createGain()
        
        osc.connect(gain)
        gain.connect(audioContext.destination)
        
        osc.frequency.setValueAtTime(freq, audioContext.currentTime)
        gain.gain.setValueAtTime(0.1, audioContext.currentTime)
        gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.2)
        
        osc.start(audioContext.currentTime + index * 0.1)
        osc.stop(audioContext.currentTime + index * 0.1 + 0.2)
      })
    } catch (error) {
      console.log('Звук достижения недоступен:', error)
    }
  }
}

/**
 * Переключает звуковые эффекты
 */
const toggleSound = () => {
  soundEnabled.value = !soundEnabled.value
  saveGameProgress()
}

/**
 * Переключает тему
 */
const toggleTheme = () => {
  darkTheme.value = !darkTheme.value
  document.body.classList.toggle('light-theme', !darkTheme.value)
  saveGameProgress()
}

/**
 * Покупка улучшения
 * @param upgrade - объект улучшения для покупки
 */
const buyUpgrade = (upgrade: any) => {
  if (score.value >= upgrade.cost) {
    score.value -= upgrade.cost
    upgrade.count++
    upgrade.cost = Math.floor(upgrade.cost * 1.5)
    
    // Применяем эффект улучшения
    switch (upgrade.effect) {
      case 'autoClick':
        // Перезапускаем авто-кликер с новым количеством
        if (autoClickInterval) clearInterval(autoClickInterval)
        autoClickInterval = setInterval(() => {
          score.value += upgrade.count * multiplier.value
        }, 1000)
        break
      case 'multiplier':
        multiplier.value += 0.5
        break
      case 'megaClick':
        multiplier.value += 7123078
        break
      case 'timeWarp':
        // Ускоряем авто-кликер
        if (autoClickInterval) {
          clearInterval(autoClickInterval)
          const autoClicker = upgrades.value.find(u => u.effect === 'autoClick')
          if (autoClicker && autoClicker.count > 0) {
            autoClickInterval = setInterval(() => {
              score.value += autoClicker.count * multiplier.value * (1 + upgrade.count * 0.5)
            }, 1000 / (1 + upgrade.count * 0.5))
          }
        }
        break
      case 'quantumBoost':
        // Квантовый буст - случайный бонус (ИСПРАВЛЕНО: было вычитание, стало сложение)
        const bonus = Math.floor(Math.random() * 100) + 1
        score.value += bonus * multiplier.value
        // Показываем уведомление о бонусе
        showBonusNotification(bonus * multiplier.value)
        break
      case 'infinityStone':
        // Камень бесконечности - огромный буст
        multiplier.value *= 2
        break
    }
    
    // Воспроизводим звук покупки
    if (soundEnabled.value) {
      try {
        const audioContext = new (window.AudioContext || (window as any).webkitAudioContext)()
        const oscillator = audioContext.createOscillator()
        const gainNode = audioContext.createGain()
        
        oscillator.connect(gainNode)
        gainNode.connect(audioContext.destination)
        
        oscillator.frequency.setValueAtTime(1000, audioContext.currentTime)
        oscillator.frequency.exponentialRampToValueAtTime(500, audioContext.currentTime + 0.2)
        
        gainNode.gain.setValueAtTime(0.1, audioContext.currentTime)
        gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.2)
        
        oscillator.start(audioContext.currentTime)
        oscillator.stop(audioContext.currentTime + 0.2)
      } catch (error) {
        console.log('Звук покупки недоступен:', error)
      }
    }
  }
}

/**
 * Показывает уведомление о бонусе
 * @param bonus - размер бонуса
 */
const showBonusNotification = (bonus: number) => {
  const notification = {
    id: Date.now(),
    text: `+${formatNumber(bonus)}`,
    style: {
      left: '50%',
      top: '50%',
      transform: 'translate(-50%, -50%)',
      animationDelay: '0s'
    }
  }
  
  // Добавляем в массив частиц для отображения
  clickParticles.value.push(notification)
  
  // Удаляем через 2 секунды
  setTimeout(() => {
    clickParticles.value = clickParticles.value.filter(p => p.id !== notification.id)
  }, 2000)
}

// ===== РАБОТА С LOCALSTORAGE =====

/**
 * Сохраняет прогресс игры в localStorage
 */
const saveGameProgress = () => {
  const gameData = {
    score: score.value,
    multiplier: multiplier.value,
    upgrades: upgrades.value,
    achievements: achievements.value,
    totalClicks: totalClicks.value,
    soundEnabled: soundEnabled.value,
    darkTheme: darkTheme.value,
    timestamp: Date.now()
  }
  localStorage.setItem('cosmicClickerProgress', JSON.stringify(gameData))
}

/**
 * Загружает прогресс игры из localStorage
 */
const loadGameProgress = () => {
  const savedData = localStorage.getItem('cosmicClickerProgress')
  if (savedData) {
    try {
      const gameData = JSON.parse(savedData)
      score.value = gameData.score || 0
      multiplier.value = gameData.multiplier || 1
      totalClicks.value = gameData.totalClicks || 0
      soundEnabled.value = gameData.soundEnabled !== undefined ? gameData.soundEnabled : true
      darkTheme.value = gameData.darkTheme !== undefined ? gameData.darkTheme : true
      
      // Восстанавливаем улучшения
      if (gameData.upgrades) {
        upgrades.value = gameData.upgrades.map((savedUpgrade: any) => {
          const originalUpgrade = upgrades.value.find(u => u.id === savedUpgrade.id)
          return {
            ...originalUpgrade,
            ...savedUpgrade
          }
        })
      }
      
      // Восстанавливаем достижения
      if (gameData.achievements) {
        achievements.value = gameData.achievements.map((savedAchievement: any) => {
          const originalAchievement = achievements.value.find(a => a.id === savedAchievement.id)
          return {
            ...originalAchievement,
            ...savedAchievement
          }
        })
      }
      
      // Применяем тему
      document.body.classList.toggle('light-theme', !darkTheme.value)
      
      console.log('Прогресс игры загружен:', gameData)
    } catch (error) {
      console.error('Ошибка при загрузке прогресса:', error)
    }
  }
}

/**
 * Сбрасывает прогресс игры
 */
const resetGameProgress = () => {
  localStorage.removeItem('cosmicClickerProgress')
  score.value = 0
  multiplier.value = 1
  totalClicks.value = 0
  clicksPerSecond.value = 0
  lastClickTime.value = 0
  soundEnabled.value = true
  darkTheme.value = true
  
  // Сбрасываем улучшения
  upgrades.value.forEach(upgrade => {
    upgrade.count = 0
    upgrade.cost = 1
  })
  
  // Сбрасываем достижения
  achievements.value.forEach((achievement, index) => {
    achievement.unlocked = index === 0 // Только первое достижение разблокировано
  })
  
  // Очищаем уведомления
  achievementNotifications.value = []
  clickParticles.value = []
  
  // Очищаем авто-кликер
  if (autoClickInterval) {
    clearInterval(autoClickInterval)
    autoClickInterval = null
  }
  
  // Сбрасываем тему
  document.body.classList.remove('light-theme')
  
  console.log('Прогресс игры сброшен')
}

// ===== НАБЛЮДАТЕЛИ =====

// Автоматически сохраняем прогресс при изменении счёта
watch(score, () => {
  saveGameProgress()
})

// Автоматически сохраняем прогресс при изменении множителя
watch(multiplier, () => {
  saveGameProgress()
})

// Автоматически сохраняем прогресс при изменении улучшений
watch(upgrades, () => {
  saveGameProgress()
}, { deep: true })

// Автоматически сохраняем прогресс при изменении достижений
watch(achievements, () => {
  saveGameProgress()
}, { deep: true })

// Автоматически сохраняем прогресс при изменении настроек звука
watch(soundEnabled, () => {
  saveGameProgress()
})

// Автоматически сохраняем прогресс при изменении темы
watch(darkTheme, () => {
  saveGameProgress()
})

// Сбрасываем CPS через секунду после последнего клика
watch(lastClickTime, () => {
  setTimeout(() => {
    if (Date.now() - lastClickTime.value >= 1000) {
      clicksPerSecond.value = 0
    }
  }, 1000)
})

// ===== ЖИЗНЕННЫЙ ЦИКЛ КОМПОНЕНТА =====

/**
 * Инициализация при монтировании компонента
 */
onMounted(() => {
  // Загружаем сохранённый прогресс
  loadGameProgress()
  
  // Восстанавливаем авто-кликер если он был куплен
  const autoClicker = upgrades.value.find(u => u.effect === 'autoClick')
  if (autoClicker && autoClicker.count > 0) {
    autoClickInterval = setInterval(() => {
      score.value += autoClicker.count * multiplier.value
    }, 1000)
  }
  
  // Добавляем обработчик для сброса прогресса (Ctrl+R)
  document.addEventListener('keydown', handleKeyDown)
  
  console.log('Космический кликер инициализирован!')
})

/**
 * Очистка при размонтировании компонента
 */
onUnmounted(() => {
  // Очищаем интервал авто-кликера
  if (autoClickInterval) {
    clearInterval(autoClickInterval)
  }
  
  // Удаляем обработчик клавиш
  document.removeEventListener('keydown', handleKeyDown)
  
  console.log('Компонент кликера размонтирован')
})

// Экспортируем функции для возможного использования в других компонентах
defineExpose({
  saveGameProgress,
  loadGameProgress,
  resetGameProgress
})
</script>

<style scoped>
/* ===== ОСНОВНОЙ КОНТЕЙНЕР ===== */
.clicker-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  position: relative;
  overflow: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  transition: all 0.3s ease;
}

.clicker-container.light-theme {
  background: linear-gradient(135deg, #74b9ff 0%, #0984e3 100%);
}

/* ===== ПАНЕЛЬ УПРАВЛЕНИЯ ===== */
.control-panel {
  position: fixed;
  top: 20px;
  right: 20px;
  display: flex;
  gap: 10px;
  z-index: 1000;
}

.control-btn {
  width: 50px;
  height: 50px;
  border: none;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.control-btn:hover {
  transform: scale(1.1);
  background: rgba(255, 255, 255, 0.3);
}

.control-btn.active {
  background: rgba(78, 205, 196, 0.3);
  border-color: rgba(78, 205, 196, 0.5);
}

.control-icon {
  font-size: 1.5rem;
}

/* ===== АНИМИРОВАННЫЕ ЧАСТИЦЫ ФОНА ===== */
.particles {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.particle {
  position: absolute;
  width: 4px;
  height: 4px;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  animation: float 6s infinite linear;
}

@keyframes float {
  0% {
    transform: translateY(100vh) rotate(0deg);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    transform: translateY(-100px) rotate(360deg);
    opacity: 0;
  }
}

/* ===== ОСНОВНОЙ КОНТЕНТ ИГРЫ ===== */
.game-content {
  position: relative;
  z-index: 10;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

/* ===== СЕКЦИЯ СЧЁТА ===== */
.score-section {
  text-align: center;
  margin-bottom: 3rem;
}

.game-title {
  font-size: 3.5rem;
  font-weight: 800;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4);
  background-size: 400% 400%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradientShift 3s ease-in-out infinite;
  margin-bottom: 1rem;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
}

@keyframes gradientShift {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

.score-display {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 1rem 2rem;
  margin-bottom: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.score-label {
  font-size: 1.2rem;
  color: rgba(255, 255, 255, 0.8);
  margin-right: 1rem;
}

.score-value {
  font-size: 2.5rem;
  font-weight: 700;
  color: #fff;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
}

.stats-row {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1rem;
}

.multiplier, .cps-display {
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  padding: 0.5rem 1.5rem;
  border-radius: 25px;
  color: white;
  font-weight: 600;
}

.total-clicks {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 1rem 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: inline-block;
}

.clicks-label {
  font-size: 1.2rem;
  color: rgba(255, 255, 255, 0.8);
  margin-right: 1rem;
}

.clicks-value {
  font-size: 2.5rem;
  font-weight: 700;
  color: #fff;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.5);
}

/* ===== СЕКЦИЯ КЛИКА ===== */
.click-section {
  display: flex;
  justify-content: center;
  margin-bottom: 3rem;
}

.click-button {
  position: relative;
  width: 200px;
  height: 200px;
  border: none;
  border-radius: 50%;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.click-button:hover {
  transform: scale(1.05);
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.4);
}

.click-button.clicked {
  transform: scale(0.95);
}

.button-content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: white;
}

.button-icon {
  font-size: 3rem;
  margin-bottom: 0.5rem;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

.button-text {
  font-size: 1.2rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 2px;
}

.button-glow {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
  border-radius: 50%;
  animation: glow 2s ease-in-out infinite alternate;
}

@keyframes glow {
  from { opacity: 0.5; }
  to { opacity: 1; }
}

/* ===== СЕКЦИЯ ДОСТИЖЕНИЙ ===== */
.achievements-section {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  margin-bottom: 2rem;
}

.achievements-title {
  text-align: center;
  color: white;
  font-size: 2rem;
  margin-bottom: 2rem;
  font-weight: 600;
}

.achievements-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
}

.achievement-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 1.5rem;
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  align-items: center;
  gap: 1rem;
  opacity: 0.6;
}

.achievement-card.unlocked {
  background: rgba(76, 175, 80, 0.2);
  border-color: rgba(76, 175, 80, 0.5);
  opacity: 1;
  transform: scale(1.02);
}

.achievement-icon {
  font-size: 2rem;
  width: 50px;
  text-align: center;
}

.achievement-info {
  flex: 1;
}

.achievement-name {
  color: white;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

.achievement-description {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
}

.achievement-status {
  font-size: 1.5rem;
}

.status-unlocked {
  color: #4caf50;
}

.status-locked {
  color: rgba(255, 255, 255, 0.5);
}

/* ===== СЕКЦИЯ УЛУЧШЕНИЙ ===== */
.upgrades-section {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.upgrades-title {
  text-align: center;
  color: white;
  font-size: 2rem;
  margin-bottom: 2rem;
  font-weight: 600;
}

.upgrades-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.upgrade-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  padding: 1.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  align-items: center;
  gap: 1rem;
}

.upgrade-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  border-color: rgba(255, 255, 255, 0.3);
}

.upgrade-card.affordable {
  background: rgba(76, 175, 80, 0.2);
  border-color: rgba(76, 175, 80, 0.5);
}

.upgrade-icon {
  font-size: 2rem;
  width: 50px;
  text-align: center;
}

.upgrade-info {
  flex: 1;
}

.upgrade-name {
  color: white;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

.upgrade-cost {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
}

.upgrade-count {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-weight: 600;
  font-size: 0.9rem;
}

/* ===== ЛЕТАЮЩИЕ ЧАСТИЦЫ ===== */
.click-particle {
  position: fixed;
  pointer-events: none;
  color: #4ecdc4;
  font-weight: 700;
  font-size: 1.2rem;
  animation: particleFloat 1s ease-out forwards;
  z-index: 1000;
}

@keyframes particleFloat {
  0% {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translateY(-100px) scale(1.5);
    opacity: 0;
  }
}

/* ===== УВЕДОМЛЕНИЯ О ДОСТИЖЕНИЯХ ===== */
.achievement-notification {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  border-radius: 15px;
  padding: 1rem 2rem;
  color: white;
  font-weight: 600;
  z-index: 1000;
  animation: slideDown 0.5s ease-out, fadeOut 0.5s ease-in 2.5s forwards;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

@keyframes slideDown {
  from {
    transform: translateX(-50%) translateY(-100px);
    opacity: 0;
  }
  to {
    transform: translateX(-50%) translateY(0);
    opacity: 1;
  }
}

@keyframes fadeOut {
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
}

.notification-content {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.notification-icon {
  font-size: 2rem;
}

.notification-text {
  text-align: left;
}

.notification-title {
  font-size: 1.1rem;
  margin-bottom: 0.25rem;
}

.notification-achievement {
  font-size: 1rem;
  opacity: 0.9;
}

/* ===== АДАПТИВНОСТЬ ===== */
@media (max-width: 768px) {
  .game-title {
    font-size: 2.5rem;
  }
  
  .click-button {
    width: 150px;
    height: 150px;
  }
  
  .button-icon {
    font-size: 2rem;
  }
  
  .upgrades-grid, .achievements-grid {
    grid-template-columns: 1fr;
  }
  
  .stats-row {
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .control-panel {
    top: 10px;
    right: 10px;
  }
  
  .control-btn {
    width: 40px;
    height: 40px;
  }
  
  .control-icon {
    font-size: 1.2rem;
  }
}
</style>
