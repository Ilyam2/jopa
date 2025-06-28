<template>
  <div class="contacts-container">
    <!-- Анимированный фон -->
    <div class="contacts-bg">
      <div class="floating-shape shape-1"></div>
      <div class="floating-shape shape-2"></div>
      <div class="floating-shape shape-3"></div>
      <div class="floating-shape shape-4"></div>
    </div>
    
    <!-- Основной контент -->
    <div class="contacts-content">
      <div class="contacts-header">
        <h1 class="contacts-title">📞 Центр Связи</h1>
        <p class="contacts-subtitle">Свяжитесь с командой Космического Кликера!</p>
      </div>

      <!-- Карточки контактов -->
      <div class="contact-grid">
        <div class="contact-card" v-for="contact in contacts" :key="contact.id">
          <div class="contact-avatar">
            <div class="avatar-icon">{{ contact.icon }}</div>
            <div class="avatar-glow"></div>
          </div>
          <div class="contact-info">
            <h3 class="contact-name">{{ contact.name }}</h3>
            <p class="contact-role">{{ contact.role }}</p>
            <div class="contact-details">
              <div class="contact-item">
                <span class="contact-label">📧</span>
                <span class="contact-value">{{ contact.email }}</span>
              </div>
              <div class="contact-item">
                <span class="contact-label">📱</span>
                <span class="contact-value">{{ contact.phone }}</span>
              </div>
            </div>
          </div>
          <div class="contact-actions">
            <button class="action-btn primary" @click="sendMessage(contact)">
              <span class="btn-icon">💬</span>
              <span class="btn-text">Сообщение</span>
            </button>
            <button class="action-btn secondary" @click="callContact(contact)">
              <span class="btn-icon">📞</span>
              <span class="btn-text">Звонок</span>
            </button>
          </div>
        </div>
      </div>

      <!-- Форма обратной связи -->
      <div class="contact-form-section">
        <h2 class="form-title">🚀 Отправить нам сообщение</h2>
        <form class="contact-form" @submit.prevent="submitForm">
          <div class="form-row">
            <div class="form-group">
              <label class="form-label">Имя</label>
              <input 
                v-model="form.name" 
                type="text" 
                class="form-input" 
                placeholder="Ваше имя"
                required
              />
            </div>
            <div class="form-group">
              <label class="form-label">Email</label>
              <input 
                v-model="form.email" 
                type="email" 
                class="form-input" 
                placeholder="ваш@email.com"
                required
              />
            </div>
          </div>
          
          <div class="form-group">
            <label class="form-label">Тема</label>
            <input 
              v-model="form.subject" 
              type="text" 
              class="form-input" 
              placeholder="О чём это?"
              required
            />
          </div>
          
          <div class="form-group">
            <label class="form-label">Сообщение</label>
            <textarea 
              v-model="form.message" 
              class="form-textarea" 
              placeholder="Расскажите, что у вас на уме..."
              rows="5"
              required
            ></textarea>
          </div>
          
          <button type="submit" class="submit-btn">
            <span class="submit-icon">🚀</span>
            <span class="submit-text">Отправить сообщение</span>
            <div class="submit-glow"></div>
          </button>
        </form>
      </div>

      <!-- Социальные сети -->
      <div class="social-section">
        <h2 class="social-title">🌐 Следите за нами</h2>
        <div class="social-links">
          <a href="#" class="social-link" v-for="social in socialLinks" :key="social.name">
            <span class="social-icon">{{ social.icon }}</span>
            <span class="social-name">{{ social.name }}</span>
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

// ===== РЕАКТИВНЫЕ ПЕРЕМЕННЫЕ =====
// Данные формы обратной связи
const form = ref({
  name: '',
  email: '',
  subject: '',
  message: ''
})

// ===== КОНФИГУРАЦИЯ КОНТАКТОВ =====
// Массив контактов команды
const contacts = ref([
  {
    id: 1,
    name: 'Алекс Джонсон',
    role: 'Ведущий разработчик',
    icon: '👨‍💻',
    email: 'alex@cosmicclicker.com',
    phone: '+1 (555) 123-4567'
  },
  {
    id: 2,
    name: 'Сара Чен',
    role: 'UI/UX дизайнер',
    icon: '🎨',
    email: 'sarah@cosmicclicker.com',
    phone: '+1 (555) 234-5678'
  },
  {
    id: 3,
    name: 'Майк Родригес',
    role: 'Гейм-дизайнер',
    icon: '🎮',
    email: 'mike@cosmicclicker.com',
    phone: '+1 (555) 345-6789'
  },
  {
    id: 4,
    name: 'Эмма Уилсон',
    role: 'Менеджер сообщества',
    icon: '🌟',
    email: 'emma@cosmicclicker.com',
    phone: '+1 (555) 456-7890'
  }
])

// ===== КОНФИГУРАЦИЯ СОЦИАЛЬНЫХ СЕТЕЙ =====
// Массив ссылок на социальные сети
const socialLinks = ref([
  { name: 'Twitter', icon: '🐦' },
  { name: 'Discord', icon: '🎮' },
  { name: 'GitHub', icon: '💻' },
  { name: 'YouTube', icon: '📺' },
  { name: 'Instagram', icon: '📸' }
])

// ===== ОСНОВНЫЕ ФУНКЦИИ =====

/**
 * Отправляет сообщение контакту
 * @param contact - объект контакта
 */
const sendMessage = (contact: any) => {
  // Здесь будет реализация отправки сообщения
  console.log('Отправка сообщения:', contact.name)
  alert(`Отправка сообщения ${contact.name}...`)
}

/**
 * Звонит контакту
 * @param contact - объект контакта
 */
const callContact = (contact: any) => {
  // Здесь будет реализация звонка
  console.log('Звонок:', contact.name)
  alert(`Звонок ${contact.name}...`)
}

/**
 * Отправляет форму обратной связи
 */
const submitForm = () => {
  // Здесь будет реализация отправки формы
  console.log('Форма отправлена:', form.value)
  
  // Показываем уведомление об успешной отправке
  alert('Сообщение успешно отправлено! Мы свяжемся с вами в ближайшее время.')
  
  // Сбрасываем форму
  form.value = {
    name: '',
    email: '',
    subject: '',
    message: ''
  }
}
</script>

<style scoped>
/* ===== ОСНОВНОЙ КОНТЕЙНЕР ===== */
.contacts-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 50%, #f093fb 100%);
  position: relative;
  overflow: hidden;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* ===== АНИМИРОВАННЫЙ ФОН ===== */
.contacts-bg {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.floating-shape {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.1);
  animation: float 10s ease-in-out infinite;
}

.shape-1 {
  width: 150px;
  height: 150px;
  top: 10%;
  left: 5%;
  animation-delay: 0s;
}

.shape-2 {
  width: 100px;
  height: 100px;
  top: 30%;
  right: 10%;
  animation-delay: 2s;
}

.shape-3 {
  width: 80px;
  height: 80px;
  bottom: 20%;
  left: 15%;
  animation-delay: 4s;
}

.shape-4 {
  width: 120px;
  height: 120px;
  bottom: 40%;
  right: 5%;
  animation-delay: 6s;
}

@keyframes float {
  0%, 100% { transform: translateY(0px) rotate(0deg); }
  50% { transform: translateY(-30px) rotate(180deg); }
}

/* ===== ОСНОВНОЙ КОНТЕНТ ===== */
.contacts-content {
  position: relative;
  z-index: 10;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
}

/* ===== ЗАГОЛОВОК ===== */
.contacts-header {
  text-align: center;
  margin-bottom: 3rem;
}

.contacts-title {
  font-size: 3rem;
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

.contacts-subtitle {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1.2rem;
  font-weight: 400;
}

/* ===== СЕТКА КОНТАКТОВ ===== */
.contact-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  margin-bottom: 4rem;
}

.contact-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.contact-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  transition: left 0.5s ease;
}

.contact-card:hover::before {
  left: 100%;
}

.contact-card:hover {
  transform: translateY(-10px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
  border-color: rgba(255, 255, 255, 0.4);
}

/* ===== АВАТАР КОНТАКТА ===== */
.contact-avatar {
  position: relative;
  width: 80px;
  height: 80px;
  margin: 0 auto 1.5rem;
}

.avatar-icon {
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2rem;
  position: relative;
  z-index: 2;
}

.avatar-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 100%;
  height: 100%;
  background: radial-gradient(circle, rgba(255, 255, 255, 0.3) 0%, transparent 70%);
  border-radius: 50%;
  animation: glow 2s ease-in-out infinite alternate;
}

@keyframes glow {
  from { opacity: 0.5; }
  to { opacity: 1; }
}

/* ===== ИНФОРМАЦИЯ О КОНТАКТЕ ===== */
.contact-info {
  text-align: center;
  margin-bottom: 1.5rem;
}

.contact-name {
  color: white;
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
}

.contact-role {
  color: rgba(255, 255, 255, 0.7);
  font-size: 1rem;
  margin-bottom: 1rem;
}

.contact-details {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  justify-content: center;
}

.contact-label {
  font-size: 1rem;
}

.contact-value {
  color: white;
  font-size: 0.9rem;
}

/* ===== КНОПКИ ДЕЙСТВИЙ ===== */
.contact-actions {
  display: flex;
  gap: 1rem;
  justify-content: center;
}

.action-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 600;
  font-size: 0.9rem;
}

.action-btn.primary {
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  color: white;
}

.action-btn.secondary {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.action-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

.btn-icon {
  font-size: 1rem;
}

/* ===== СЕКЦИЯ ФОРМЫ ===== */
.contact-form-section {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
  margin-bottom: 3rem;
}

.form-title {
  text-align: center;
  color: white;
  font-size: 2rem;
  margin-bottom: 2rem;
  font-weight: 600;
}

.contact-form {
  max-width: 600px;
  margin: 0 auto;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-label {
  display: block;
  color: white;
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.form-input,
.form-textarea {
  width: 100%;
  padding: 1rem;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.form-input::placeholder,
.form-textarea::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
  border-color: #4ecdc4;
  box-shadow: 0 0 20px rgba(78, 205, 196, 0.3);
}

.form-textarea {
  resize: vertical;
  min-height: 120px;
}

/* ===== КНОПКА ОТПРАВКИ ===== */
.submit-btn {
  position: relative;
  width: 100%;
  padding: 1rem 2rem;
  border: none;
  border-radius: 25px;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4);
  color: white;
  font-size: 1.1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  overflow: hidden;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.3);
}

.submit-glow {
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transition: left 0.5s ease;
}

.submit-btn:hover .submit-glow {
  left: 100%;
}

.submit-icon {
  font-size: 1.2rem;
}

/* ===== СЕКЦИЯ СОЦИАЛЬНЫХ СЕТЕЙ ===== */
.social-section {
  text-align: center;
}

.social-title {
  color: white;
  font-size: 2rem;
  margin-bottom: 2rem;
  font-weight: 600;
}

.social-links {
  display: flex;
  justify-content: center;
  gap: 2rem;
  flex-wrap: wrap;
}

.social-link {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  text-decoration: none;
  color: white;
  transition: all 0.3s ease;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.social-link:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.2);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.social-icon {
  font-size: 2rem;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-5px); }
}

.social-name {
  font-weight: 600;
  font-size: 0.9rem;
}

/* ===== АДАПТИВНОСТЬ ===== */
@media (max-width: 768px) {
  .contacts-title {
    font-size: 2.5rem;
  }
  
  .form-row {
    grid-template-columns: 1fr;
  }
  
  .contact-actions {
    flex-direction: column;
  }
  
  .social-links {
    gap: 1rem;
  }
  
  .social-link {
    padding: 0.75rem;
  }
}
</style>
