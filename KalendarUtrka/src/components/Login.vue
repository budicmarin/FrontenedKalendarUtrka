<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{ inModal?: boolean }>()

const emit = defineEmits<{
  (e: 'login-success', token: string, username: string): void
  (e: 'go-to-register'): void
}>()

const email= ref('')
const password = ref('')
const errorMsg = ref('')
const isLoading = ref(false)
const showPassword = ref(false)

// Promijeni URL ako backend sluša na drugom portu
const API_BASE = 'http://localhost:3000'

async function handleLogin() {
  errorMsg.value = ''

  if (!email.value.trim() || !password.value) {
    errorMsg.value = 'Molimo unesite email i lozinku.'
    return
  }

  isLoading.value = true

  try {
    const response = await fetch(`${API_BASE}/api/users/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        email: email.value.trim(),
        password: password.value,
      }),
    })

    const data = await response.json()

    if (!response.ok) {
      errorMsg.value = data.message || 'Pogrešno korisničko ime ili lozinka.'
      return
    }

    // Spremi token u localStorage
    localStorage.setItem('auth_token', data.token)
    localStorage.setItem('auth_username', data.username ?? email.value.trim())

    emit('login-success', data.token, data.username ?? email.value.trim())
  } catch (err) {
    errorMsg.value = 'Nije moguće spojiti se na server. Provjeri je li backend pokrenut.'
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="login-page">
    <!-- Background -->
    <div class="bg-glow bg-glow--red"></div>
    <div class="bg-glow bg-glow--purple"></div>

    <!-- Card -->
    <div class="login-card">
      <!-- Logo / brand -->
      <div class="brand">
        <div class="brand-icon">🏁</div>
        <h1 class="brand-title">Kalendar Utrka</h1>
        <p class="brand-subtitle">Prijavite se kako biste vidjeli kalendar</p>
      </div>

      <!-- Form -->
      <form class="login-form" @submit.prevent="handleLogin" id="login-form" novalidate>
        <!-- Username -->
        <div class="field">
          <label class="field-label" for="login-email">Email</label>
          <div class="field-wrap">
            <span class="field-icon">👤</span>
            <input
              id="login-email"
              v-model="email"
              type="text"
              class="field-input"
              placeholder="Unesite email"
              autocomplete="email"
              :disabled="isLoading"
            />
          </div>
        </div>

        <!-- Password -->
        <div class="field">
          <label class="field-label" for="login-password">Lozinka</label>
          <div class="field-wrap">
            <span class="field-icon">🔒</span>
            <input
              id="login-password"
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              class="field-input"
              placeholder="Unesite lozinku"
              autocomplete="current-password"
              :disabled="isLoading"
            />
            <button
              type="button"
              class="toggle-pw"
              @click="showPassword = !showPassword"
              :aria-label="showPassword ? 'Sakrij lozinku' : 'Prikaži lozinku'"
              id="toggle-password"
            >
              {{ showPassword ? '🙈' : '👁️' }}
            </button>
          </div>
        </div>

        <!-- Error message -->
        <Transition name="fade">
          <div v-if="errorMsg" class="error-msg" role="alert" id="login-error">
            <span class="error-icon">⚠️</span>
            {{ errorMsg }}
          </div>
        </Transition>

        <!-- Submit -->
        <button
          type="submit"
          class="btn-submit"
          :disabled="isLoading"
          id="login-submit"
        >
          <span v-if="isLoading" class="spinner"></span>
          <span v-if="!isLoading">Prijava</span>
          <span v-else>Prijava u tijeku...</span>
        </button>
      </form>

      <!-- Footer -->
      <p v-if="!props.inModal" class="login-footer">
        Nemaš račun?
        <button type="button" class="link-btn" @click="emit('go-to-register')" id="go-to-register">
          Registriraj se
        </button>
      </p>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=Orbitron:wght@700;900&display=swap');

* { box-sizing: border-box; }

/* ── Page ── */
.login-page {
  min-height: 100vh;
  background: #080b14;
  font-family: 'Inter', sans-serif;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  position: relative;
  overflow: hidden;
}

/* ── Background glows ── */
.bg-glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(120px);
  pointer-events: none;
  opacity: 0.35;
}

.bg-glow--red {
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, #dc2626 0%, transparent 70%);
  top: -180px;
  left: -120px;
}

.bg-glow--purple {
  width: 500px;
  height: 500px;
  background: radial-gradient(circle, #7c3aed 0%, transparent 70%);
  bottom: -180px;
  right: -120px;
}

/* ── Card ── */
.login-card {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 420px;
  background: rgba(15, 23, 42, 0.85);
  border: 1px solid rgba(255, 255, 255, 0.09);
  border-radius: 24px;
  padding: 44px 40px 36px;
  backdrop-filter: blur(20px);
  box-shadow:
    0 0 0 1px rgba(255,255,255,0.04),
    0 40px 80px rgba(0,0,0,0.5);
}

/* ── Brand ── */
.brand {
  text-align: center;
  margin-bottom: 36px;
}

.brand-icon {
  font-size: 3rem;
  line-height: 1;
  margin-bottom: 14px;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50%       { transform: translateY(-6px); }
}

.brand-title {
  font-family: 'Orbitron', sans-serif;
  font-size: 1.7rem;
  font-weight: 900;
  background: linear-gradient(135deg, #fff 0%, #f87171 55%, #c026d3 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0 0 8px;
}

.brand-subtitle {
  font-size: 0.875rem;
  color: #64748b;
  margin: 0;
}

/* ── Form ── */
.login-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 7px;
}

.field-label {
  font-size: 0.8rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.07em;
}

.field-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.field-icon {
  position: absolute;
  left: 14px;
  font-size: 1rem;
  pointer-events: none;
  user-select: none;
}

.field-input {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 13px 44px 13px 42px;
  color: #e2e8f0;
  font-size: 0.95rem;
  font-family: 'Inter', sans-serif;
  outline: none;
  transition: border-color 0.2s, background 0.2s, box-shadow 0.2s;
}

.field-input::placeholder { color: #334155; }

.field-input:focus {
  border-color: rgba(220, 38, 38, 0.6);
  background: rgba(255, 255, 255, 0.07);
  box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.12);
}

.field-input:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.toggle-pw {
  position: absolute;
  right: 12px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  padding: 4px;
  color: #64748b;
  transition: opacity 0.2s;
  line-height: 1;
}

.toggle-pw:hover { opacity: 0.75; }

/* ── Error ── */
.error-msg {
  background: rgba(220, 38, 38, 0.1);
  border: 1px solid rgba(220, 38, 38, 0.3);
  border-radius: 10px;
  padding: 11px 14px;
  font-size: 0.875rem;
  color: #fca5a5;
  display: flex;
  align-items: center;
  gap: 8px;
}

.error-icon { flex-shrink: 0; }

/* ── Submit button ── */
.btn-submit {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%);
  border: none;
  border-radius: 12px;
  color: #fff;
  font-size: 1rem;
  font-weight: 700;
  font-family: 'Inter', sans-serif;
  cursor: pointer;
  transition: opacity 0.2s, transform 0.15s, box-shadow 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  box-shadow: 0 4px 20px rgba(220, 38, 38, 0.35);
  margin-top: 4px;
}

.btn-submit:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
  box-shadow: 0 8px 28px rgba(220, 38, 38, 0.45);
}

.btn-submit:active:not(:disabled) {
  transform: translateY(0);
}

.btn-submit:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* ── Spinner ── */
.spinner {
  width: 18px;
  height: 18px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  flex-shrink: 0;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* ── Footer ── */
.login-footer {
  text-align: center;
  font-size: 0.8rem;
  color: #334155;
  margin: 22px 0 0;
}

/* ── Fade transition ── */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.25s, transform 0.25s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}

/* ── Responsive ── */
@media (max-width: 480px) {
  .login-card { padding: 32px 24px 28px; }
  .brand-title { font-size: 1.4rem; }
}
</style>
