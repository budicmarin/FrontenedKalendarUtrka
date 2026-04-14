<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Login from './components/Login.vue'
import RaceCalendar from './components/RaceCalendar.vue'

const isLoggedIn = ref(false)
const loggedUser = ref('')
const showLoginModal = ref(false)

onMounted(() => {
  const token = localStorage.getItem('auth_token')
  const username = localStorage.getItem('auth_username')
  if (token && username) {
    isLoggedIn.value = true
    loggedUser.value = username
  }
})

function onLoginSuccess(_token: string, username: string) {
  isLoggedIn.value = true
  loggedUser.value = username
  showLoginModal.value = false
}

function handleLogout() {
  localStorage.removeItem('auth_token')
  localStorage.removeItem('auth_username')
  isLoggedIn.value = false
  loggedUser.value = ''
}

function openLogin() {
  showLoginModal.value = true
}

function closeLogin() {
  showLoginModal.value = false
}
</script>

<template>
  <!-- Navbar -->
  <nav class="navbar">
    <div class="navbar-brand">
      <span class="brand-emoji">🏁</span>
      <span class="brand-text">Kalendar Utrka</span>
    </div>

    <div class="navbar-right">
      <!-- Nije prijavljen -->
      <template v-if="!isLoggedIn">
        <button class="btn-nav-login" @click="openLogin" id="open-login-btn">
          <span>🔑</span> Prijava
        </button>
      </template>

      <!-- Prijavljen -->
      <template v-else>
        <span class="navbar-user">
          <span class="user-dot"></span>
          {{ loggedUser }}
        </span>
        <button class="btn-logout" @click="handleLogout" id="logout-btn">
          Odjava
        </button>
      </template>
    </div>
  </nav>

  <!-- Uvijek prikazuj kalendar -->
  <RaceCalendar />

  <!-- Login modal -->
  <Transition name="modal">
    <div
      v-if="showLoginModal"
      class="modal-backdrop"
      @click.self="closeLogin"
      id="login-modal-backdrop"
    >
      <div class="modal-wrapper">
        <button class="modal-x" @click="closeLogin" id="close-login-modal" aria-label="Zatvori">✕</button>
        <Login @login-success="onLoginSuccess" />
      </div>
    </div>
  </Transition>
</template>

<style>
/* Global reset */
*, *::before, *::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  width: 100%;
  min-height: 100vh;
  background: #080b14;
  overflow-x: hidden;
}

#app {
  width: 100%;
  min-height: 100vh;
}

/* ── Navbar ── */
.navbar {
  position: sticky;
  top: 0;
  z-index: 40;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 28px;
  height: 62px;
  background: rgba(8, 11, 20, 0.88);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  box-shadow: 0 1px 30px rgba(0, 0, 0, 0.4);
}

.navbar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  text-decoration: none;
}

.brand-emoji {
  font-size: 1.4rem;
  line-height: 1;
}

.brand-text {
  font-family: 'Orbitron', 'Inter', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  color: #f1f5f9;
  letter-spacing: 0.04em;
  background: linear-gradient(90deg, #fff 0%, #f87171 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.navbar-right {
  display: flex;
  align-items: center;
  gap: 14px;
}

/* Prijava gumb */
.btn-nav-login {
  display: flex;
  align-items: center;
  gap: 7px;
  background: linear-gradient(135deg, rgba(220,38,38,0.18) 0%, rgba(192,38,211,0.18) 100%);
  border: 1px solid rgba(220, 38, 38, 0.4);
  color: #f87171;
  padding: 9px 20px;
  border-radius: 10px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: background 0.2s, border-color 0.2s, box-shadow 0.2s, transform 0.15s;
}

.btn-nav-login:hover {
  background: linear-gradient(135deg, rgba(220,38,38,0.28) 0%, rgba(192,38,211,0.28) 100%);
  border-color: rgba(220, 38, 38, 0.65);
  box-shadow: 0 4px 18px rgba(220, 38, 38, 0.25);
  transform: translateY(-1px);
}

/* Korisnik */
.navbar-user {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.875rem;
  font-weight: 500;
  color: #94a3b8;
  font-family: 'Inter', sans-serif;
}

.user-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #22c55e;
  box-shadow: 0 0 8px rgba(34, 197, 94, 0.6);
  flex-shrink: 0;
}

/* Odjava gumb */
.btn-logout {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #64748b;
  padding: 8px 16px;
  border-radius: 9px;
  font-size: 0.82rem;
  font-weight: 600;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: all 0.2s;
}

.btn-logout:hover {
  background: rgba(220, 38, 38, 0.12);
  border-color: rgba(220, 38, 38, 0.35);
  color: #f87171;
}

/* ── Modal backdrop ── */
.modal-backdrop {
  position: fixed;
  inset: 0;
  z-index: 100;
  background: rgba(0, 0, 0, 0.72);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.modal-wrapper {
  position: relative;
  width: 100%;
  max-width: 460px;
}

.modal-x {
  position: absolute;
  top: 16px;
  right: 16px;
  z-index: 10;
  background: rgba(255, 255, 255, 0.07);
  border: 1px solid rgba(255, 255, 255, 0.12);
  color: #64748b;
  width: 34px;
  height: 34px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 0.85rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s, color 0.2s;
  font-family: sans-serif;
}

.modal-x:hover {
  background: rgba(220, 38, 38, 0.2);
  color: #f87171;
}

/* ── Modal transition ── */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.25s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-wrapper,
.modal-leave-active .modal-wrapper {
  transition: transform 0.25s ease;
}

.modal-enter-from .modal-wrapper {
  transform: scale(0.93) translateY(16px);
}

.modal-leave-to .modal-wrapper {
  transform: scale(0.93) translateY(16px);
}
</style>
