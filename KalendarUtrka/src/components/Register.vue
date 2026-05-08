<script setup lang="ts">
import { ref, computed } from 'vue'

const emit = defineEmits<{
  (e: 'register-success'): void
  (e: 'go-to-login'): void
}>()

const firstName = ref('')
const lastName = ref('')
const username = ref('')
const email = ref('')
const password = ref('')
const confirmPassword = ref('')
const showPassword = ref(false)
const showConfirmPassword = ref(false)
const errorMsg = ref('')
const successMsg = ref('')
const isLoading = ref(false)

const API_BASE = 'https://backendkalendarutrka.onrender.com'

const passwordStrength = computed(() => {
  const p = password.value
  if (!p) return 0
  let score = 0
  if (p.length >= 8) score++
  if (/[A-Z]/.test(p)) score++
  if (/[0-9]/.test(p)) score++
  if (/[^A-Za-z0-9]/.test(p)) score++
  return score
})

const strengthLabel = computed(() => ['', 'Slaba', 'Osrednja', 'Dobra', 'Jaka'][passwordStrength.value] ?? '')
const strengthClass = computed(() => ['', 'weak', 'fair', 'good', 'strong'][passwordStrength.value] ?? '')

const passwordsMatch = computed(() =>
  confirmPassword.value === '' || password.value === confirmPassword.value
)

async function handleRegister() {
  errorMsg.value = ''
  successMsg.value = ''

  if (!firstName.value.trim() || !lastName.value.trim() || !username.value.trim() || !email.value.trim() || !password.value) {
    errorMsg.value = 'Molimo popunite sva obavezna polja.'
    return
  }
  if (!passwordsMatch.value) {
    errorMsg.value = 'Lozinke se ne podudaraju.'
    return
  }
  if (passwordStrength.value < 2) {
    errorMsg.value = 'Lozinka je preslaba. Koristite barem 8 znakova s brojevima ili velikim slovima.'
    return
  }
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value.trim())) {
    errorMsg.value = 'Molimo unesite ispravnu email adresu.'
    return
  }

  isLoading.value = true
  try {
    const response = await fetch(`${API_BASE}/api/users/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        name: firstName.value.trim(),
        surname: lastName.value.trim(),
        username: username.value.trim(),
        email: email.value.trim(),
        password: password.value,
      }),
    })
    const data = await response.json()
    if (!response.ok) {
      errorMsg.value = data.message || 'Registracija nije uspjela. Pokušajte ponovo.'
      return
    }
    successMsg.value = 'Registracija uspješna! Preusmjeravanje na prijavu...'
    setTimeout(() => emit('go-to-login'), 1800)
  } catch {
    errorMsg.value = 'Nije moguće spojiti se na server.'
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="register-page">
    <div class="bg-glow bg-glow--blue"></div>
    <div class="bg-glow bg-glow--purple"></div>

    <div class="register-card">
      <div class="brand">
        <div class="brand-icon">🏁</div>
        <h1 class="brand-title">Registracija</h1>
        <p class="brand-subtitle">Kreirajte novi račun</p>
      </div>

      <form class="form" @submit.prevent="handleRegister" novalidate>

        <!-- Ime i Prezime -->
        <div class="field-row">
          <div class="field">
            <label class="field-label" for="reg-firstname">Ime <span class="req">*</span></label>
            <div class="field-wrap">
              <span class="field-icon">👤</span>
              <input id="reg-firstname" v-model="firstName" type="text" class="field-input"
                placeholder="Vaše ime" autocomplete="given-name" :disabled="isLoading" />
            </div>
          </div>
          <div class="field">
            <label class="field-label" for="reg-lastname">Prezime <span class="req">*</span></label>
            <div class="field-wrap">
              <span class="field-icon">👤</span>
              <input id="reg-lastname" v-model="lastName" type="text" class="field-input"
                placeholder="Vaše prezime" autocomplete="family-name" :disabled="isLoading" />
            </div>
          </div>
        </div>

        <!-- Korisničko ime -->
        <div class="field">
          <label class="field-label" for="reg-username">Korisničko ime <span class="req">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🏷️</span>
            <input id="reg-username" v-model="username" type="text" class="field-input"
              placeholder="Odaberite korisničko ime" autocomplete="username" :disabled="isLoading" />
          </div>
        </div>

        <!-- Email -->
        <div class="field">
          <label class="field-label" for="reg-email">Email <span class="req">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">✉️</span>
            <input id="reg-email" v-model="email" type="email" class="field-input"
              placeholder="vasa@email.com" autocomplete="email" :disabled="isLoading" />
          </div>
        </div>

        <!-- Lozinka -->
        <div class="field">
          <label class="field-label" for="reg-password">Lozinka <span class="req">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🔒</span>
            <input id="reg-password" v-model="password" :type="showPassword ? 'text' : 'password'"
              class="field-input" placeholder="Minimalno 8 znakova" autocomplete="new-password" :disabled="isLoading" />
            <button type="button" class="toggle-pw" @click="showPassword = !showPassword"
              :aria-label="showPassword ? 'Sakrij lozinku' : 'Prikaži lozinku'">
              {{ showPassword ? '🙈' : '👁️' }}
            </button>
          </div>
          <Transition name="fade">
            <div v-if="password" class="strength-wrap">
              <div class="strength-bars">
                <div v-for="i in 4" :key="i" :class="['strength-bar', i <= passwordStrength ? strengthClass : '']"></div>
              </div>
              <span :class="['strength-text', strengthClass]">{{ strengthLabel }}</span>
            </div>
          </Transition>
        </div>

        <!-- Potvrda lozinke -->
        <div class="field">
          <label class="field-label" for="reg-confirm">Potvrdi lozinku <span class="req">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🔒</span>
            <input id="reg-confirm" v-model="confirmPassword"
              :type="showConfirmPassword ? 'text' : 'password'"
              :class="['field-input', { 'input-error': confirmPassword && !passwordsMatch }]"
              placeholder="Ponovite lozinku" autocomplete="new-password" :disabled="isLoading" />
            <button type="button" class="toggle-pw" @click="showConfirmPassword = !showConfirmPassword"
              :aria-label="showConfirmPassword ? 'Sakrij' : 'Prikaži'">
              {{ showConfirmPassword ? '🙈' : '👁️' }}
            </button>
          </div>
          <Transition name="fade">
            <p v-if="confirmPassword && !passwordsMatch" class="field-hint error">⚠️ Lozinke se ne podudaraju</p>
            <p v-else-if="confirmPassword && passwordsMatch" class="field-hint ok">✅ Lozinke se podudaraju</p>
          </Transition>
        </div>

        <!-- Messages -->
        <Transition name="fade">
          <div v-if="errorMsg" class="error-msg" role="alert"><span>⚠️</span> {{ errorMsg }}</div>
        </Transition>
        <Transition name="fade">
          <div v-if="successMsg" class="success-msg" role="status"><span>🎉</span> {{ successMsg }}</div>
        </Transition>

        <button type="submit" class="btn-submit" :disabled="isLoading || !passwordsMatch">
          <span v-if="isLoading" class="spinner"></span>
          <span>{{ isLoading ? 'Registracija...' : 'Registriraj se' }}</span>
        </button>
      </form>

      <p class="switch-link">
        Već imaš račun?
        <button type="button" class="link-btn" @click="emit('go-to-login')">Prijavi se</button>
      </p>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;700;800&family=DM+Sans:wght@300;400;500;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

.register-page {
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
  opacity: .28;
}

.bg-glow--blue {
  width: min(500px, 90vw); height: min(500px, 90vw);
  background: radial-gradient(circle, #2563eb 0%, transparent 70%);
  top: -180px; right: -100px;
}

.bg-glow--purple {
  width: min(500px, 90vw); height: min(500px, 90vw);
  background: radial-gradient(circle, #7c3aed 0%, transparent 70%);
  bottom: -180px; left: -120px;
}

/* ── Card ── */
.register-card {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 500px;
  background: rgba(15,23,42,.88);
  border: 1px solid rgba(255,255,255,.09);
  border-radius: 24px;
  padding: clamp(24px, 5vw, 44px) clamp(20px, 6vw, 40px);
  backdrop-filter: blur(20px);
  box-shadow: 0 0 0 1px rgba(255,255,255,.04), 0 40px 80px rgba(0,0,0,.5);
}

/* ── Brand ── */
.brand {
  text-align: center;
  margin-bottom: 28px;
}

.brand-icon {
  font-size: clamp(2rem, 6vw, 2.8rem);
  line-height: 1;
  margin-bottom: 12px;
  display: block;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%,100% { transform: translateY(0); }
  50%      { transform: translateY(-5px); }
}

.brand-title {
  font-family: 'Syne', sans-serif;
  font-size: clamp(1.35rem, 5vw, 1.7rem);
  font-weight: 800;
  background: linear-gradient(135deg, #fff 0%, #60a5fa 55%, #c026d3 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 6px;
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
  gap: 15px;
}

.field-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.field-label {
  font-size: .73rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: .08em;
}

.req { color: #f87171; }

.field-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.field-icon {
  position: absolute;
  left: 13px;
  font-size: .9rem;
  pointer-events: none;
}

.field-input {
  width: 100%;
  background: rgba(255,255,255,.05);
  border: 1px solid rgba(255,255,255,.1);
  border-radius: 11px;
  padding: 12px 42px 12px 40px;
  color: #e2e8f0;
  font-size: .88rem;
  font-family: 'DM Sans', sans-serif;
  outline: none;
  transition: border-color .2s, background .2s, box-shadow .2s;
}

.field-input::placeholder { color: #334155; }

.field-input:focus {
  border-color: rgba(96,165,250,.55);
  background: rgba(255,255,255,.07);
  box-shadow: 0 0 0 3px rgba(96,165,250,.1);
}

.field-input:disabled { opacity: .5; cursor: not-allowed; }
.field-input.input-error { border-color: rgba(220,38,38,.6) !important; }

.toggle-pw {
  position: absolute;
  right: 11px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: .9rem;
  padding: 4px;
  color: #64748b;
  line-height: 1;
  transition: opacity .2s;
}

.toggle-pw:hover { opacity: .7; }

/* ── Strength ── */
.strength-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 4px;
}

.strength-bars {
  display: flex;
  gap: 4px;
  flex: 1;
}

.strength-bar {
  flex: 1;
  height: 4px;
  border-radius: 2px;
  background: rgba(255,255,255,.08);
  transition: background .3s;
}

.strength-bar.weak   { background: #ef4444; }
.strength-bar.fair   { background: #f59e0b; }
.strength-bar.good   { background: #3b82f6; }
.strength-bar.strong { background: #22c55e; }

.strength-text {
  font-size: .7rem;
  font-weight: 600;
  min-width: 52px;
  text-align: right;
}

.strength-text.weak   { color: #ef4444; }
.strength-text.fair   { color: #f59e0b; }
.strength-text.good   { color: #3b82f6; }
.strength-text.strong { color: #22c55e; }

/* ── Hints ── */
.field-hint {
  font-size: .76rem;
  font-weight: 500;
  margin-top: 2px;
}

.field-hint.error { color: #f87171; }
.field-hint.ok    { color: #4ade80; }

/* ── Messages ── */
.error-msg, .success-msg {
  border-radius: 10px;
  padding: 11px 14px;
  font-size: .875rem;
  display: flex;
  align-items: center;
  gap: 8px;
}

.error-msg {
  background: rgba(220,38,38,.1);
  border: 1px solid rgba(220,38,38,.3);
  color: #fca5a5;
}

.success-msg {
  background: rgba(34,197,94,.1);
  border: 1px solid rgba(34,197,94,.3);
  color: #4ade80;
}

/* ── Submit ── */
.btn-submit {
  width: 100%;
  padding: 13px;
  background: linear-gradient(135deg, #2563eb, #7c3aed);
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
  box-shadow: 0 4px 20px rgba(37,99,235,.35);
  transition: opacity .2s, transform .15s, box-shadow .2s;
  margin-top: 4px;
}

.btn-submit:hover:not(:disabled) {
  opacity: .9;
  transform: translateY(-1px);
  box-shadow: 0 8px 28px rgba(37,99,235,.45);
}

.btn-submit:disabled { opacity: .5; cursor: not-allowed; }

/* ── Spinner ── */
.spinner {
  width: 17px; height: 17px;
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
  margin-top: 20px;
}

.link-btn {
  background: none;
  border: none;
  color: #60a5fa;
  font-size: inherit;
  font-family: 'DM Sans', sans-serif;
  font-weight: 600;
  cursor: pointer;
  padding: 0 2px;
  transition: color .2s;
}

.link-btn:hover { color: #93c5fd; }

/* ── Transition ── */
.fade-enter-active, .fade-leave-active { transition: opacity .2s, transform .2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-4px); }

/* ── Responsive ── */
@media (max-width: 480px) {
  .field-row { grid-template-columns: 1fr; }
  .register-card { border-radius: 20px; }
  .field-input { font-size: .86rem; }
}
</style>