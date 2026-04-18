<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Login from './components/Login.vue'
import Register from './components/Register.vue'
import RaceCalendar from './components/RaceCalendar.vue'
import AddRace from './components/AddRace.vue'

type ModalView = 'none' | 'login' | 'register' | 'add-race'

const isLoggedIn = ref(false)
const loggedUser = ref('')
const modalView = ref<ModalView>('none')

// 1. Tipizacija je sada ispravna
const loggedUserId = ref<string | undefined>(undefined)

onMounted(() => {
  const token = localStorage.getItem('auth_token')
  const username = localStorage.getItem('auth_username')
  
  // 2. VAŽNO: localStorage.getItem vraća 'string | null'. 
  // Moramo dodati '?? undefined' da bi TS bio sretan.
  const userId = localStorage.getItem('auth_userId') ?? undefined
  
  if (token && username) {
    isLoggedIn.value = true
    loggedUser.value = username
    loggedUserId.value = userId
  }
})

function onLoginSuccess(_token: string, username: string, userId: string) {
  isLoggedIn.value = true
  loggedUser.value = username
  loggedUserId.value = userId
  modalView.value = 'none'
}

function handleLogout() {
  localStorage.removeItem('auth_token')
  localStorage.removeItem('auth_username')
  localStorage.removeItem('auth_userId')
  
  isLoggedIn.value = false
  loggedUser.value = ''
  
  // 3. Umjesto praznog stringa '', koristimo undefined 
  // kako bi se podudaralo s početnim stanjem.
  loggedUserId.value = undefined
}

function closeModal() {
  modalView.value = 'none'
}

function onRegisterSuccess() {
  modalView.value = 'login'
}
</script>

<template>
  <nav class="navbar">
    <div class="navbar-brand">
      <span class="brand-emoji">🏁</span>
      <span class="brand-text">Kalendar Utrka</span>
    </div>

    <div class="navbar-right">
      <template v-if="!isLoggedIn">
        <button class="btn-nav btn-nav--ghost" @click="modalView = 'register'" id="open-register-btn">
          Registracija
        </button>
        <button class="btn-nav btn-nav--primary" @click="modalView = 'login'" id="open-login-btn">
          <span>🔑</span> Prijava
        </button>
      </template>

      <template v-else>
        <span class="navbar-user">
          <span class="user-dot"></span>
          {{ loggedUser }}
        </span>
        <button class="btn-logout" @click="handleLogout" id="logout-btn">
          Odjava
        </button>
        <button class="btn-nav btn-nav--primary" @click="modalView = 'add-race'" id="open-add-race-btn">
          <span>+</span> Dodaj utrku
        </button>
      </template>
    </div>
  </nav>

  <RaceCalendar :userId="loggedUserId" />

  <Transition name="modal">
    <div v-if="modalView !== 'none'" class="modal-backdrop" @click.self="closeModal" id="auth-modal-backdrop">
      <div class="modal-wrapper">
        <button class="modal-x" @click="closeModal" id="close-modal-btn" aria-label="Zatvori">✕</button>

        <div class="modal-tabs" v-if="modalView === 'login' || modalView === 'register'">
          <button :class="['modal-tab', { active: modalView === 'login' }]" @click="modalView = 'login'" id="tab-login">
            Prijava
          </button>
          <button :class="['modal-tab', { active: modalView === 'register' }]" @click="modalView = 'register'" id="tab-register">
            Registracija
          </button>
        </div>

        <Transition name="slide" mode="out-in">
          <Login
            v-if="modalView === 'login'"
            key="login"
            @login-success="onLoginSuccess"
            @go-to-register="modalView = 'register'"
            :in-modal="true"
          />
          <Register
            v-else-if="modalView === 'register'"
            key="register"
            @register-success="onRegisterSuccess"
            @go-to-login="modalView = 'login'"
            :in-modal="true"
          />
          <AddRace
            v-else-if="modalView === 'add-race'"
            key="add-race"
            @race-added="closeModal"
          />
        </Transition>
      </div>
    </div>
  </Transition>
</template>

<style>

/* ── Global reset ── */
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
  background: rgba(8, 11, 20, 0.9);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  box-shadow: 0 1px 30px rgba(0, 0, 0, 0.4);
  font-family: 'Inter', sans-serif;
}

.navbar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.brand-emoji { font-size: 1.4rem; line-height: 1; }

.brand-text {
  font-family: 'Orbitron', 'Inter', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  background: linear-gradient(90deg, #fff 0%, #f87171 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  letter-spacing: 0.04em;
}

.navbar-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

/* Nav gumbi */
.btn-nav {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 9px 20px;
  border-radius: 10px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: all 0.2s;
  white-space: nowrap;
}

.btn-nav--ghost {
  background: transparent;
  border: 1px solid rgba(255, 255, 255, 0.12);
  color: #94a3b8;
}
.btn-nav--ghost:hover {
  background: rgba(255, 255, 255, 0.06);
  color: #e2e8f0;
  border-color: rgba(255, 255, 255, 0.2);
}

.btn-nav--primary {
  background: linear-gradient(135deg, rgba(220,38,38,0.2), rgba(192,38,211,0.2));
  border: 1px solid rgba(220, 38, 38, 0.4);
  color: #f87171;
}
.btn-nav--primary:hover {
  background: linear-gradient(135deg, rgba(220,38,38,0.3), rgba(192,38,211,0.3));
  border-color: rgba(220, 38, 38, 0.65);
  box-shadow: 0 4px 18px rgba(220, 38, 38, 0.25);
  transform: translateY(-1px);
}

/* Korisnik online */
.navbar-user {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.875rem;
  font-weight: 500;
  color: #94a3b8;
}

.user-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #22c55e;
  box-shadow: 0 0 8px rgba(34, 197, 94, 0.6);
}

.btn-logout {
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.1);
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
  background: rgba(220,38,38,0.12);
  border-color: rgba(220,38,38,0.35);
  color: #f87171;
}

/* ── Modal backdrop ── */
.modal-backdrop {
  position: fixed;
  inset: 0;
  z-index: 100;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.modal-wrapper {
  position: relative;
  width: 100%;
  max-width: 500px;
  background: rgba(10, 15, 28, 0.98);
  border: 1px solid rgba(255,255,255,0.09);
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 40px 100px rgba(0,0,0,0.7);
}

/* Zatvaranje */
.modal-x {
  position: absolute;
  top: 16px;
  right: 16px;
  z-index: 10;
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.12);
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
}
.modal-x:hover {
  background: rgba(220,38,38,0.2);
  color: #f87171;
}

/* ── Tabs ── */
.modal-tabs {
  display: flex;
  border-bottom: 1px solid rgba(255,255,255,0.07);
}

.modal-tab {
  flex: 1;
  padding: 16px;
  background: none;
  border: none;
  font-size: 0.9rem;
  font-weight: 600;
  font-family: 'Inter', sans-serif;
  color: #475569;
  cursor: pointer;
  transition: color 0.2s, background 0.2s;
  position: relative;
}

.modal-tab::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 20%;
  right: 20%;
  height: 2px;
  background: linear-gradient(90deg, #dc2626, #c026d3);
  border-radius: 2px;
  transform: scaleX(0);
  transition: transform 0.25s;
}

.modal-tab.active {
  color: #f1f5f9;
}
.modal-tab.active::after {
  transform: scaleX(1);
}

/* Inner cards: uklonimo vanjski padding/background jer modal-wrapper to daje */
.modal-wrapper :deep(.login-page),
.modal-wrapper :deep(.register-page) {
  min-height: unset;
  background: transparent;
  padding: 0;
}

.modal-wrapper :deep(.bg-glow) {
  display: none;
}

.modal-wrapper :deep(.login-card),
.modal-wrapper :deep(.register-card) {
  max-width: 100%;
  border-radius: 0;
  border: none;
  box-shadow: none;
  backdrop-filter: none;
  background: transparent;
  padding: 32px 36px 28px;
}

.modal-wrapper :deep(.brand) {
  display: none; /* brand je vidljiv u tabovima */
}

.modal-wrapper :deep(.login-footer),
.modal-wrapper :deep(.switch-link) {
  display: none; /* switcher je u tabovima */
}

/* ── Modal transitions ── */
.modal-enter-active, .modal-leave-active { transition: opacity 0.25s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; }
.modal-enter-active .modal-wrapper { transition: transform 0.25s ease; }
.modal-enter-from .modal-wrapper { transform: scale(0.94) translateY(14px); }

/* Slide između login/register */
.slide-enter-active, .slide-leave-active { transition: opacity 0.2s, transform 0.2s; }
.slide-enter-from { opacity: 0; transform: translateX(20px); }
.slide-leave-to   { opacity: 0; transform: translateX(-20px); }
</style>
