<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps<{ inModal?: boolean }>()

const emit = defineEmits<{
  (e: 'login-success', token: string, username: string, userId: string): void
  (e: 'go-to-register'): void
}>()

const email = ref('')
const password = ref('')
const errorMsg = ref('')
const isLoading = ref(false)
const showPassword = ref(false)

const API_BASE = 'https://backendkalendarutrka.onrender.com'

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
      body: JSON.stringify({ email: email.value.trim(), password: password.value }),
    })
    const data = await response.json()
    if (!response.ok) {
      errorMsg.value = data.message || 'Pogrešno korisničko ime ili lozinka.'
      return
    }
    localStorage.setItem('auth_token', data.token)
    localStorage.setItem('auth_username', data.username ?? email.value.trim())
    localStorage.setItem('auth_userId', data.id)
    emit('login-success', data.token, data.username ?? email.value.trim(), data.id)
  } catch {
    errorMsg.value = 'Nije moguće spojiti se na server.'
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="login-page">
    <div class="bg-glow bg-glow--red"></div>
    <div class="bg-glow bg-glow--purple"></div>

    <div class="login-card">
      <div class="brand">
        <div class="brand-icon">🏁</div>
        <h1 class="brand-title">Kalendar Utrka</h1>
        <p class="brand-subtitle">Prijavite se u svoj račun</p>
      </div>

      <form class="form" @submit.prevent="handleLogin" novalidate>
        <div class="field">
          <label class="field-label" for="login-email">Email</label>
          <div class="field-wrap">
            <span class="field-icon">✉️</span>
            <input
              id="login-email"
              v-model="email"
              type="email"
              class="field-input"
              placeholder="vasa@email.com"
              autocomplete="email"
              :disabled="isLoading"
            />
          </div>
        </div>

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
            >{{ showPassword ? '🙈' : '👁️' }}</button>
          </div>
        </div>

        <Transition name="fade">
          <div v-if="errorMsg" class="error-msg" role="alert">
            <span>⚠️</span> {{ errorMsg }}
          </div>
        </Transition>

        <button type="submit" class="btn-submit" :disabled="isLoading">
          <span v-if="isLoading" class="spinner"></span>
          <span>{{ isLoading ? 'Prijava u tijeku...' : 'Prijava' }}</span>
        </button>
      </form>

      <p v-if="!props.inModal" class="switch-link">
        Nemaš račun?
        <button type="button" class="link-btn" @click="emit('go-to-register')">Registriraj se</button>
      </p>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;700;800&family=DM+Sans:wght@300;400;500;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

.login-page {
  min-height: 100vh;
  background: #080b14;
  font-family: 'DM Sans', sans-serif;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(16px, 5vw, 32px);
  position: relative;
  overflow: hidden;
}

/* ── Glows ── */
.bg-glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(130px);
  pointer-events: none;
  opacity: .3;
}

.bg-glow--red {
  width: min(500px, 90vw); height: min(500px, 90vw);
  background: radial-gradient(circle, #dc2626 0%, transparent 70%);
  top: -180px; left: -120px;
}

.bg-glow--purple {
  width: min(500px, 90vw); height: min(500px, 90vw);
  background: radial-gradient(circle, #7c3aed 0%, transparent 70%);
  bottom: -180px; right: -120px;
}

/* ── Card ── */
.login-card {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 420px;
  background: rgba(15,23,42,.88);
  border: 1px solid rgba(255,255,255,.09);
  border-radius: 24px;
  padding: clamp(28px, 6vw, 48px) clamp(22px, 6vw, 40px);
  backdrop-filter: blur(20px);
  box-shadow: 0 0 0 1px rgba(255,255,255,.04), 0 40px 80px rgba(0,0,0,.5);
}

/* ── Brand ── */
.brand {
  text-align: center;
  margin-bottom: 36px;
}

.brand-icon {
  font-size: clamp(2.2rem, 6vw, 3rem);
  line-height: 1;
  margin-bottom: 14px;
  display: block;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%,100% { transform: translateY(0); }
  50%      { transform: translateY(-6px); }
}

.brand-title {
  font-family: 'Syne', sans-serif;
  font-size: clamp(1.4rem, 5vw, 1.8rem);
  font-weight: 800;
  background: linear-gradient(135deg, #fff 0%, #f87171 55%, #c026d3 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 8px;
}

.brand-subtitle {
  font-size: .875rem;
  color: #64748b;
  font-weight: 300;
}

/* ── Form ── */
.form {
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
  font-size: .75rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: .08em;
}

.field-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.field-icon {
  position: absolute;
  left: 14px;
  font-size: .95rem;
  pointer-events: none;
}

.field-input {
  width: 100%;
  background: rgba(255,255,255,.05);
  border: 1px solid rgba(255,255,255,.1);
  border-radius: 12px;
  padding: 13px 44px 13px 42px;
  color: #e2e8f0;
  font-size: .9rem;
  font-family: 'DM Sans', sans-serif;
  outline: none;
  transition: border-color .2s, background .2s, box-shadow .2s;
}

.field-input::placeholder { color: #334155; }

.field-input:focus {
  border-color: rgba(220,38,38,.55);
  background: rgba(255,255,255,.07);
  box-shadow: 0 0 0 3px rgba(220,38,38,.1);
}

.field-input:disabled { opacity: .5; cursor: not-allowed; }

.toggle-pw {
  position: absolute;
  right: 12px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: .95rem;
  padding: 4px;
  color: #64748b;
  line-height: 1;
  transition: opacity .2s;
}

.toggle-pw:hover { opacity: .7; }

/* ── Error ── */
.error-msg {
  background: rgba(220,38,38,.1);
  border: 1px solid rgba(220,38,38,.3);
  border-radius: 10px;
  padding: 11px 14px;
  font-size: .875rem;
  color: #fca5a5;
  display: flex;
  align-items: center;
  gap: 8px;
}

/* ── Submit ── */
.btn-submit {
  width: 100%;
  padding: 14px;
  background: linear-gradient(135deg, #dc2626, #b91c1c);
  border: none;
  border-radius: 12px;
  color: #fff;
  font-size: .95rem;
  font-weight: 700;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  box-shadow: 0 4px 20px rgba(220,38,38,.35);
  transition: opacity .2s, transform .15s, box-shadow .2s;
  margin-top: 4px;
}

.btn-submit:hover:not(:disabled) {
  opacity: .9;
  transform: translateY(-1px);
  box-shadow: 0 8px 28px rgba(220,38,38,.45);
}

.btn-submit:disabled { opacity: .6; cursor: not-allowed; }

/* ── Spinner ── */
.spinner {
  width: 18px; height: 18px;
  border: 2px solid rgba(255,255,255,.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin .7s linear infinite;
  flex-shrink: 0;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* ── Footer ── */
.switch-link {
  text-align: center;
  font-size: .82rem;
  color: #334155;
  margin-top: 24px;
}

.link-btn {
  background: none;
  border: none;
  color: #f87171;
  font-size: inherit;
  font-family: 'DM Sans', sans-serif;
  font-weight: 600;
  cursor: pointer;
  padding: 0 2px;
  transition: color .2s;
}

.link-btn:hover { color: #fca5a5; }

/* ── Transition ── */
.fade-enter-active, .fade-leave-active { transition: opacity .25s, transform .25s; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-6px); }

/* ── Responsive ── */
@media (max-width: 440px) {
  .login-card { border-radius: 20px; }
  .field-input { font-size: .88rem; }
}
</style>