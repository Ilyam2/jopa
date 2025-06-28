<template>
  <div class="user-clicker-container">
    <!-- Анимированный фон -->
    <div class="animated-bg">
      <div class="bg-circle circle-1"></div>
      <div class="bg-circle circle-2"></div>
      <div class="bg-circle circle-3"></div>
    </div>
    
    <!-- Основной контент -->
    <div class="user-content">
      <div class="user-header">
        <h1 class="user-title">🎮 Кликер Пользователя</h1>
        <div class="user-stats">
          <div class="stat-card">
            <div class="stat-icon">💎</div>
            <div class="stat-info">
              <div class="stat-label">Клики</div>
              <div class="stat-value">{{ formatNumber(clickCount) }}</div>
            </div>
          </div>
          <div class="stat-card">
            <div class="stat-icon">⚡</div>
            <div class="stat-info">
              <div class="stat-label">Сила</div>
              <div class="stat-value">{{ powerLevel }}</div>
            </div>
          </div>
        </div>
      </div>

      <!-- Основная область клика -->
      <div class="click-area">
        <div class="click-zone">
          <button 
            @click="handleClick" 
            class="user-click-button"
            :class="{ 'clicked': isClicked }"
            @mousedown="isClicked = true"
            @mouseup="isClicked = false"
            @mouseleave="isClicked = false"
          >
            <div class="click-button-inner">
              <div class="click-icon">🎯</div>
              <div class="click-text">БАМ!</div>
              <div class="click-power">+{{ clickPower }}</div>
            </div>
            <div class="click-ripple" v-if="showRipple"></div>
          </button>
        </div>
        
        <!-- Уведомления о достижениях -->
        <div class="achievements">
          <div v-for="achievement in achievements" :key="achievement.id" class="achievement-badge">
            <div class="achievement-icon">{{ achievement.icon }}</div>
            <div class="achievement-text">{{ achievement.name }}</div>
          </div>
        </div>
      </div>

      <!-- Секция усилений -->
      <div class="powerups-section">
        <h2 class="powerups-title">🔥 Усиления</h2>
        <div class="powerups-grid">
          <div 
            v-for="powerup in powerups" 
            :key="powerup.id"
            class="powerup-card"
            :class="{ 'available': clickCount >= powerup.cost }"
            @click="buyPowerup(powerup)"
          >
            <div class="powerup-icon">{{ powerup.icon }}</div>
            <div class="powerup-details">
              <div class="powerup-name">{{ powerup.name }}</div>
              <div class="powerup-description">{{ powerup.description }}</div>
              <div class="powerup-cost">{{ formatNumber(powerup.cost) }} кликов</div>
            </div>
            <div class="powerup-level">{{ powerup.level }}/{{ powerup.maxLevel }}</div>
          </div>
        </div>
      </div>

      <!-- Летающие частицы -->
      <div v-for="particle in particles" :key="particle.id" class="floating-particle" :style="particle.style">
        +{{ particle.value }}
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

// ===== РЕАКТИВНЫЕ ПЕРЕМЕННЫЕ =====
// Счётчик кликов игрока
const clickCount = ref<number>(0)
// Сила каждого клика
const clickPower = ref<number>(1)
// Уровень силы игрока
const powerLevel = ref<number>(1)
// Состояние нажатия кнопки (для анимации)
const isClicked = ref<boolean>(false)
// Показывать ли ripple эффект
const showRipple = ref<boolean>(false)
// Массив частиц для визуальных эффектов
const particles = ref<any[]>([])
// Уникальный ID для частиц
let particleId = 0

// ===== КОНФИГУРАЦИЯ УСИЛЕНИЙ =====
// Массив всех доступных усилений
const powerups = ref([
  { 
    id: 1, 
    name: 'Усилитель кликов', 
    icon: '⚡', 
    description: 'Удваивает силу ваших кликов',
    cost: 10, 
    level: 0, 
    maxLevel: 5,
    effect: 'amplify'
  },
  { 
    id: 2, 
    name: 'Авто-кликер', 
    icon: '🤖', 
    description: 'Автоматически кликает за вас',
    cost: 50, 
    level: 0, 
    maxLevel: 3,
    effect: 'auto'
  },
  { 
    id: 3, 
    name: 'Критический удар', 
    icon: '💥', 
    description: 'Шанс нанести критический урон',
    cost: 100, 
    level: 0, 
    maxLevel: 3,
    effect: 'critical'
  },
  { 
    id: 4, 
    name: 'Искривление времени', 
    icon: '⏰', 
    description: 'Ускоряет авто-кликер',
    cost: 250, 
    level: 0, 
    maxLevel: 2,
    effect: 'timewarp'
  }
])

// ===== КОНФИГУРАЦИЯ ДОСТИЖЕНИЙ =====
// Массив достижений игрока
const achievements = ref([
  { id: 1, name: 'Первый клик!', icon: '🎯', unlocked: true },
  { id: 2, name: 'Мастер кликов', icon: '👑', unlocked: false },
  { id: 3, name: 'Сильный игрок', icon: '💪', unlocked: false }
])

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
const createParticle = (x: number, y: number, value: number) => {
  const particle = {
    id: particleId++,
    value: value,
    style: {
      left: x + 'px',
      top: y + 'px',
      animationDelay: '0s'
    }
  }
  particles.value.push(particle)
  
  // Удаляем частицу через 1.5 секунды
  setTimeout(() => {
    particles.value = particles.value.filter(p => p.id !== particle.id)
  }, 1500)
}

// ===== ОСНОВНЫЕ ФУНКЦИИ ИГРЫ =====

/**
 * Обработчик клика по главной кнопке
 * @param event - событие мыши
 */
const handleClick = (event: MouseEvent) => {
  clickCount.value += clickPower.value
  powerLevel.value = Math.floor(clickCount.value / 100) + 1
  
  // Создаём эффект частицы
  createParticle(event.clientX, event.clientY, clickPower.value)
  
  // Показываем ripple эффект
  showRipple.value = true
  setTimeout(() => {
    showRipple.value = false
  }, 300)
  
  // Проверяем достижения
  checkAchievements()
}

/**
 * Покупка усиления
 * @param powerup - объект усиления для покупки
 */
const buyPowerup = (powerup: any) => {
  if (clickCount.value >= powerup.cost && powerup.level < powerup.maxLevel) {
    clickCount.value -= powerup.cost
    powerup.level++
    powerup.cost = Math.floor(powerup.cost * 2)
    
    // Применяем эффект усиления
    switch (powerup.effect) {
      case 'amplify':
        clickPower.value *= 2
        break
      case 'auto':
        // Логика авто-кликера будет добавлена позже
        console.log('Авто-кликер куплен!')
        break
      case 'critical':
        // Логика критических ударов будет добавлена позже
        console.log('Критический удар куплен!')
        break
      case 'timewarp':
        // Логика искривления времени будет добавлена позже
        console.log('Искривление времени куплено!')
        break
    }
  }
}

/**
 * Проверяет и разблокирует достижения
 */
const checkAchievements = () => {
  // Достижение "Мастер кликов" - 100 кликов
  if (clickCount.value >= 100 && !achievements.value[1].unlocked) {
    achievements.value[1].unlocked = true
    console.log('Достижение разблокировано: Мастер кликов!')
  }
  
  // Достижение "Сильный игрок" - 5 уровень силы
  if (powerLevel.value >= 5 && !achievements.value[2].unlocked) {
    achievements.value[2].unlocked = true
    console.log('Достижение разблокировано: Сильный игрок!')
  }
}

// ===== РАБОТА С LOCALSTORAGE =====

/**
 * Сохраняет прогресс игры в localStorage
 */
const saveGameProgress = () => {
  const gameData = {
    clickCount: clickCount.value,
    clickPower: clickPower.value,
    powerLevel: powerLevel.value,
    powerups: powerups.value,
    achievements: achievements.value,
    timestamp: Date.now()
  }
  localStorage.setItem('userClickerProgress', JSON.stringify(gameData))
  console.log('Прогресс сохранён:', gameData)
}

/**
 * Загружает прогресс игры из localStorage
 */
const loadGameProgress = () => {
  const savedData = localStorage.getItem('userClickerProgress')
  if (savedData) {
    try {
      const gameData = JSON.parse(savedData)
      
      // Восстанавливаем основные данные
      clickCount.value = gameData.clickCount || 0
      clickPower.value = gameData.clickPower || 1
      powerLevel.value = gameData.powerLevel || 1
      
      // Восстанавливаем усиления
      if (gameData.powerups) {
        powerups.value = gameData.powerups.map((savedPowerup: any) => {
          const originalPowerup = powerups.value.find(p => p.id === savedPowerup.id)
          return {
            ...originalPowerup,
            ...savedPowerup
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
  localStorage.removeItem('userClickerProgress')
  clickCount.value = 0
  clickPower.value = 1
  powerLevel.value = 1
  
  // Сбрасываем усиления
  powerups.value.forEach(powerup => {
    powerup.level = 0
    powerup.cost = powerup.id === 1 ? 10 : powerup.id === 2 ? 50 : powerup.id === 3 ? 100 : 250
  })
  
  // Сбрасываем достижения
  achievements.value.forEach((achievement, index) => {
    achievement.unlocked = index === 0 // Только первое достижение разблокировано
  })
  
  console.log('Прогресс игры сброшен')
}

// ===== НАБЛЮДАТЕЛИ =====

// Автоматически сохраняем прогресс при изменении счёта кликов
watch(clickCount, () => {
  saveGameProgress()
})

// Автоматически сохраняем прогресс при изменении силы клика
watch(clickPower, () => {
  saveGameProgress()
})

// Автоматически сохраняем прогресс при изменении уровня силы
watch(powerLevel, () => {
  saveGameProgress()
})

// Автоматически сохраняем прогресс при изменении усилений
watch(powerups, () => {
  saveGameProgress()
}, { deep: true })

// Автоматически сохраняем прогресс при изменении достижений
watch(achievements, () => {
  saveGameProgress()
}, { deep: true })

// ===== ЖИЗНЕННЫЙ ЦИКЛ КОМПОНЕНТА =====

/**
 * Инициализация при монтировании компонента
 */
onMounted(() => {
  // Загружаем сохранённый прогресс
  loadGameProgress()
  
  // Добавляем обработчик для сброса прогресса (Ctrl+Shift+R)
  const handleKeyDown = (event: KeyboardEvent) => {
    if (event.ctrlKey && event.shiftKey && event.key === 'R') {
      event.preventDefault()
      if (confirm('Сбросить весь прогресс игры?')) {
        resetGameProgress()
      }
    }
  }
  
  document.addEventListener('keydown', handleKeyDown)
  
  console.log('Кликер пользователя инициализирован!')
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
.user-clicker-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  position: relative;
  overflow: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* ===== АНИМИРОВАННЫЙ ФОН ===== */
.animated-bg {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.bg-circle {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  animation: float 8s ease-in-out infinite;
}

.circle-1 {
  width: 200px;
  height: 200px;
  top: 10%;
  left: 10%;
  animation-delay: 0s;
}

.circle-2 {
  width: 150px;
  height: 150px;
  top: 60%;
  right: 15%;
  animation-delay: 2s;
}

.circle-3 {
  width: 100px;
  height: 100px;
  bottom: 20%;
  left: 20%;
  animation-delay: 4s;
}

@keyframes float {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  50% { transform: translateY(-20px) rotate(180deg); }
}

/* ===== ОСНОВНОЙ КОНТЕНТ ===== */
.user-content {
  position: relative;
  z-index: 10;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

/* ===== ЗАГОЛОВОК И СТАТИСТИКА ===== */
.user-header {
  text-align: center;
  margin-bottom: 3rem;
}

.user-title {
  font-size: 3rem;
  font-weight: 800;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4);
  background-size: 400% 400%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradientShift 3s ease-in-out infinite;
  margin-bottom: 2rem;
  text-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
}

@keyframes gradientShift {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

.user-stats {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 1.5rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  gap: 1rem;
  min-width: 150px;
}

.stat-icon {
  font-size: 2rem;
  width: 50px;
  text-align: center;
}

.stat-info {
  flex: 1;
}

.stat-label {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  margin-bottom: 0.25rem;
}

.stat-value {
  color: white;
  font-size: 1.5rem;
  font-weight: 700;
}

/* ===== ОБЛАСТЬ КЛИКА ===== */
.click-area {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 3rem;
}

.click-zone {
  position: relative;
  margin-bottom: 2rem;
}

.user-click-button {
  position: relative;
  width: 180px;
  height: 180px;
  border: none;
  border-radius: 50%;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.3);
  overflow: hidden;
}

.user-click-button:hover {
  transform: scale(1.05);
  box-shadow: 0 20px 45px rgba(0, 0, 0, 0.4);
}

.user-click-button.clicked {
  transform: scale(0.95);
}

.click-button-inner {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: white;
}

.click-icon {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.click-text {
  font-size: 1rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin-bottom: 0.25rem;
}

.click-power {
  font-size: 0.9rem;
  opacity: 0.9;
}

.click-ripple {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  animation: ripple 0.3s ease-out;
}

@keyframes ripple {
  to {
    width: 300px;
    height: 300px;
    opacity: 0;
  }
}

/* ===== ДОСТИЖЕНИЯ ===== */
.achievements {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

.achievement-badge {
  background: rgba(255, 255, 255, 0.2);
  border-radius: 15px;
  padding: 0.75rem 1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  animation: slideIn 0.5s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.achievement-icon {
  font-size: 1.2rem;
}

.achievement-text {
  color: white;
  font-weight: 600;
  font-size: 0.9rem;
}

/* ===== СЕКЦИЯ УСИЛЕНИЙ ===== */
.powerups-section {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.powerups-title {
  text-align: center;
  color: white;
  font-size: 2rem;
  margin-bottom: 2rem;
  font-weight: 600;
}

.powerups-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
}

.powerup-card {
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

.powerup-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  border-color: rgba(255, 255, 255, 0.3);
}

.powerup-card.available {
  background: rgba(76, 175, 80, 0.2);
  border-color: rgba(76, 175, 80, 0.5);
}

.powerup-icon {
  font-size: 2rem;
  width: 50px;
  text-align: center;
}

.powerup-details {
  flex: 1;
}

.powerup-name {
  color: white;
  font-weight: 600;
  margin-bottom: 0.25rem;
}

.powerup-description {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.85rem;
  margin-bottom: 0.5rem;
}

.powerup-cost {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9rem;
  font-weight: 500;
}

.powerup-level {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-weight: 600;
  font-size: 0.9rem;
}

/* ===== ЛЕТАЮЩИЕ ЧАСТИЦЫ ===== */
.floating-particle {
  position: fixed;
  pointer-events: none;
  color: #4ecdc4;
  font-weight: 700;
  font-size: 1.2rem;
  animation: particleFloat 1.5s ease-out forwards;
  z-index: 1000;
}

@keyframes particleFloat {
  0% {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translateY(-150px) scale(1.5);
    opacity: 0;
  }
}

/* ===== АДАПТИВНОСТЬ ===== */
@media (max-width: 768px) {
  .user-title {
    font-size: 2.5rem;
  }
  
  .user-stats {
    flex-direction: column;
    align-items: center;
  }
  
  .user-click-button {
    width: 150px;
    height: 150px;
  }
  
  .click-icon {
    font-size: 2rem;
  }
  
  .powerups-grid {
    grid-template-columns: 1fr;
  }
}
</style>
