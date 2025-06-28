<template>
  <header class="modern-header">
    <div class="header-content">
      <div class="logo-section">
        <h1 class="logo">🚀 Космический Кликер</h1>
        <div class="logo-glow"></div>
      </div>
      
      <nav class="navigation">
        <NuxtLink to="/" class="nav-link" active-class="active">
          <span class="nav-icon">🏠</span>
          <span class="nav-text">Главная</span>
        </NuxtLink>
        
        <NuxtLink to="/contacts" class="nav-link" active-class="active">
          <span class="nav-icon">📞</span>
          <span class="nav-text">Контакты</span>
        </NuxtLink>
        
        <NuxtLink to="/users/add" class="nav-link" active-class="active">
          <span class="nav-icon">🎮</span>
          <span class="nav-text">Кликер</span>
        </NuxtLink>
      </nav>
      
      <div class="header-stats">
        <div class="stat-item">
          <span class="stat-icon">⭐</span>
          <span class="stat-value">0</span>
        </div>
        <div class="stat-item">
          <span class="stat-icon">⚡</span>
          <span class="stat-value">1x</span>
        </div>
      </div>
    </div>
    
    <!-- Анимированные элементы фона -->
    <div class="header-bg">
      <div class="bg-line line-1"></div>
      <div class="bg-line line-2"></div>
      <div class="bg-line line-3"></div>
    </div>
  </header>
</template>

<style scoped>
/* ===== ОСНОВНОЙ ХЕДЕР ===== */
.modern-header {
  background: linear-gradient(135deg, rgba(102, 126, 234, 0.95) 0%, rgba(118, 75, 162, 0.95) 100%);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  position: relative;
  overflow: hidden;
  z-index: 1000;
}

/* ===== АНИМИРОВАННЫЙ ФОН ===== */
.header-bg {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
}

.bg-line {
  position: absolute;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  animation: slideLine 8s linear infinite;
}

.line-1 {
  top: 20%;
  width: 100%;
  animation-delay: 0s;
}

.line-2 {
  top: 50%;
  width: 80%;
  left: 10%;
  animation-delay: 2s;
}

.line-3 {
  top: 80%;
  width: 60%;
  left: 20%;
  animation-delay: 4s;
}

@keyframes slideLine {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    transform: translateX(100%);
    opacity: 0;
  }
}

/* ===== ОСНОВНОЕ СОДЕРЖИМОЕ ХЕДЕРА ===== */
.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem 2rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  position: relative;
  z-index: 10;
}

/* ===== СЕКЦИЯ ЛОГОТИПА ===== */
.logo-section {
  position: relative;
}

.logo {
  font-size: 1.8rem;
  font-weight: 800;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96ceb4);
  background-size: 400% 400%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradientShift 3s ease-in-out infinite;
  margin: 0;
  text-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
}

@keyframes gradientShift {
  0%, 100% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
}

.logo-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  height: 100%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.2) 0%, transparent 70%);
  border-radius: 50%;
  animation: glow 2s ease-in-out infinite alternate;
}

@keyframes glow {
  from { opacity: 0.3; }
  to { opacity: 0.6; }
}

/* ===== НАВИГАЦИЯ ===== */
.navigation {
  display: flex;
  gap: 2rem;
  align-items: center;
}

.nav-link {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.25rem;
  padding: 0.75rem 1rem;
  border-radius: 15px;
  text-decoration: none;
  color: rgba(255, 255, 255, 0.8);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.nav-link::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s ease;
}

.nav-link:hover::before {
  left: 100%;
}

.nav-link:hover {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.nav-link.active {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.nav-icon {
  font-size: 1.5rem;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-3px); }
}

.nav-text {
  font-size: 0.8rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1px;
}

/* ===== СТАТИСТИКА В ХЕДЕРЕ ===== */
.header-stats {
  display: flex;
  gap: 1rem;
  align-items: center;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(255, 255, 255, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
}

.stat-item:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: scale(1.05);
}

.stat-icon {
  font-size: 1.2rem;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); }
}

.stat-value {
  color: white;
  font-weight: 700;
  font-size: 0.9rem;
}

/* ===== АДАПТИВНОСТЬ ===== */
@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    gap: 1rem;
    padding: 1rem;
  }
  
  .logo {
    font-size: 1.5rem;
  }
  
  .navigation {
    gap: 1rem;
  }
  
  .nav-link {
    padding: 0.5rem 0.75rem;
  }
  
  .nav-icon {
    font-size: 1.2rem;
  }
  
  .nav-text {
    font-size: 0.7rem;
  }
  
  .header-stats {
    gap: 0.5rem;
  }
  
  .stat-item {
    padding: 0.4rem 0.8rem;
  }
}

@media (max-width: 480px) {
  .navigation {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .nav-link {
    min-width: 80px;
  }
}
</style>