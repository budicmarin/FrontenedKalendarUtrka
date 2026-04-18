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

const API_BASE = 'https://backendkalendarutrka.onrender.com/'

// ── Validacija lozinke ──
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

const strengthLabel = computed(() => {
  const labels = ['', 'Slaba', 'Osrednja', 'Dobra', 'Jaka']
  return labels[passwordStrength.value] ?? ''
})

const strengthClass = computed(() => {
  const classes = ['', 'weak', 'fair', 'good', 'strong']
  return classes[passwordStrength.value] ?? ''
})

const passwordsMatch = computed(() =>
  confirmPassword.value === '' || password.value === confirmPassword.value
)

async function handleRegister() {
  errorMsg.value = ''
  successMsg.value = ''

  // Klijentska validacija
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

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(email.value.trim())) {
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

    successMsg.value = 'Registracija uspješna! Možete se sada prijaviti.'
    setTimeout(() => emit('go-to-login'), 1800)
  } catch (err) {
    errorMsg.value = 'Nije moguće spojiti se na server. Provjeri je li backend pokrenut.'
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
      <!-- Brand -->
      <div class="brand">
        <div class="brand-icon">🏁</div>
        <h1 class="brand-title">Registracija</h1>
        <p class="brand-subtitle">Kreirajte novi račun</p>
      </div>

      <!-- Form -->
      <form class="register-form" @submit.prevent="handleRegister" id="register-form" novalidate>

        <!-- Ime i Prezime (red) -->
        <div class="field-row">
          <div class="field">
            <label class="field-label" for="reg-firstname">Ime <span class="required">*</span></label>
            <div class="field-wrap">
              <span class="field-icon">👤</span>
              <input
                id="reg-firstname"
                v-model="firstName"
                type="text"
                class="field-input"
                placeholder="Vaše ime"
                autocomplete="given-name"
                :disabled="isLoading"
              />
            </div>
          </div>

          <div class="field">
            <label class="field-label" for="reg-lastname">Prezime <span class="required">*</span></label>
            <div class="field-wrap">
              <span class="field-icon">👤</span>
              <input
                id="reg-lastname"
                v-model="lastName"
                type="text"
                class="field-input"
                placeholder="Vaše prezime"
                autocomplete="family-name"
                :disabled="isLoading"
              />
            </div>
          </div>
        </div>

        <!-- Korisničko ime -->
        <div class="field">
          <label class="field-label" for="reg-username">Korisničko ime <span class="required">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🏷️</span>
            <input
              id="reg-username"
              v-model="username"
              type="text"
              class="field-input"
              placeholder="Odaberite korisničko ime"
              autocomplete="username"
              :disabled="isLoading"
            />
          </div>
        </div>

        <!-- Email -->
        <div class="field">
          <label class="field-label" for="reg-email">Email <span class="required">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">✉️</span>
            <input
              id="reg-email"
              v-model="email"
              type="email"
              class="field-input"
              placeholder="vasa@email.com"
              autocomplete="email"
              :disabled="isLoading"
            />
          </div>
        </div>

        <!-- Lozinka -->
        <div class="field">
          <label class="field-label" for="reg-password">Lozinka <span class="required">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🔒</span>
            <input
              id="reg-password"
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              class="field-input"
              placeholder="Minimalno 8 znakova"
              autocomplete="new-password"
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

          <!-- Strength indicator -->
          <div v-if="password" class="strength-wrap">
            <div class="strength-bars">
              <div
                v-for="i in 4"
                :key="i"
                :class="['strength-bar', i <= passwordStrength ? strengthClass : '']"
              ></div>
            </div>
            <span :class="['strength-text', strengthClass]">{{ strengthLabel }}</span>
          </div>
        </div>

        <!-- Potvrda lozinke -->
        <div class="field">
          <label class="field-label" for="reg-confirm">Potvrdi lozinku <span class="required">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🔒</span>
            <input
              id="reg-confirm"
              v-model="confirmPassword"
              :type="showConfirmPassword ? 'text' : 'password'"
              :class="['field-input', { 'input-error': !passwordsMatch }]"
              placeholder="Ponovite lozinku"
              autocomplete="new-password"
              :disabled="isLoading"
            />
            <button
              type="button"
              class="toggle-pw"
              @click="showConfirmPassword = !showConfirmPassword"
              :aria-label="showConfirmPassword ? 'Sakrij lozinku' : 'Prikaži lozinku'"
              id="toggle-confirm-password"
            >
              {{ showConfirmPassword ? '🙈' : '👁️' }}
            </button>
          </div>
          <Transition name="fade">
            <p v-if="confirmPassword && !passwordsMatch" class="field-hint error">
              ⚠️ Lozinke se ne podudaraju
            </p>
            <p v-else-if="confirmPassword && passwordsMatch" class="field-hint ok">
              ✅ Lozinke se podudaraju
            </p>
          </Transition>
        </div>

        <!-- Error poruka -->
        <Transition name="fade">
          <div v-if="errorMsg" class="error-msg" role="alert" id="register-error">
            <span>⚠️</span> {{ errorMsg }}
          </div>
        </Transition>

        <!-- Success poruka -->
        <Transition name="fade">
          <div v-if="successMsg" class="success-msg" role="status" id="register-success">
            <span>🎉</span> {{ successMsg }}
          </div>
        </Transition>

        <!-- Submit -->
        <button
          type="submit"
          class="btn-submit"
          :disabled="isLoading || !passwordsMatch"
          id="register-submit"
        >
          <span v-if="isLoading" class="spinner"></span>
          <span>{{ isLoading ? 'Registracija...' : 'Registriraj se' }}</span>
        </button>
      </form>

      <!-- Link na login -->
      <p class="switch-link">
        Već imaš račun?
        <button type="button" class="link-btn" @click="emit('go-to-login')" id="go-to-login">
          Prijavi se
        </button>
      </p>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=Orbitron:wght@700;900&display=swap');

* { box-sizing: border-box; }

.register-page {
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

/* Glows */
.bg-glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(120px);
  pointer-events: none;
  opacity: 0.3;
}
.bg-glow--blue {
  width: 500px; height: 500px;
  background: radial-gradient(circle, #2563eb 0%, transparent 70%);
  top: -180px; right: -100px;
}
.bg-glow--purple {
  width: 500px; height: 500px;
  background: radial-gradient(circle, #7c3aed 0%, transparent 70%);
  bottom: -180px; left: -120px;
}

/* Card */
.register-card {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 500px;
  background: rgba(15, 23, 42, 0.88);
  border: 1px solid rgba(255, 255, 255, 0.09);
  border-radius: 24px;
  padding: 40px 40px 32px;
  backdrop-filter: blur(20px);
  box-shadow: 0 0 0 1px rgba(255,255,255,0.04), 0 40px 80px rgba(0,0,0,0.5);
}

/* Brand */
.brand {
  text-align: center;
  margin-bottom: 30px;
}
.brand-icon {
  font-size: 2.6rem;
  line-height: 1;
  margin-bottom: 12px;
  animation: float 3s ease-in-out infinite;
}
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50%       { transform: translateY(-5px); }
}
.brand-title {
  font-family: 'Orbitron', sans-serif;
  font-size: 1.6rem;
  font-weight: 900;
  background: linear-gradient(135deg, #fff 0%, #60a5fa 55%, #c026d3 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0 0 6px;
}
.brand-subtitle {
  font-size: 0.875rem;
  color: #64748b;
  margin: 0;
}

/* Form */
.register-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.field-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.field-label {
  font-size: 0.78rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.07em;
}

.required { color: #f87171; }

.field-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.field-icon {
  position: absolute;
  left: 13px;
  font-size: 0.95rem;
  pointer-events: none;
  user-select: none;
}

.field-input {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 11px;
  padding: 12px 42px 12px 40px;
  color: #e2e8f0;
  font-size: 0.9rem;
  font-family: 'Inter', sans-serif;
  outline: none;
  transition: border-color 0.2s, background 0.2s, box-shadow 0.2s;
}

.field-input::placeholder { color: #334155; }

.field-input:focus {
  border-color: rgba(96, 165, 250, 0.6);
  background: rgba(255, 255, 255, 0.07);
  box-shadow: 0 0 0 3px rgba(96, 165, 250, 0.1);
}

.field-input:disabled { opacity: 0.5; cursor: not-allowed; }

.field-input.input-error {
  border-color: rgba(220, 38, 38, 0.6) !important;
}

.toggle-pw {
  position: absolute;
  right: 11px;
  background: none;
  border: none;
  cursor: pointer;
  font-size: 0.95rem;
  padding: 4px;
  color: #64748b;
  transition: opacity 0.2s;
  line-height: 1;
}
.toggle-pw:hover { opacity: 0.7; }

/* Strength */
.strength-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 6px;
}

.strength-bars {
  display: flex;
  gap: 5px;
  flex: 1;
}

.strength-bar {
  flex: 1;
  height: 4px;
  border-radius: 2px;
  background: rgba(255, 255, 255, 0.08);
  transition: background 0.3s;
}

.strength-bar.weak   { background: #ef4444; }
.strength-bar.fair   { background: #f59e0b; }
.strength-bar.good   { background: #3b82f6; }
.strength-bar.strong { background: #22c55e; }

.strength-text {
  font-size: 0.72rem;
  font-weight: 600;
  min-width: 52px;
  text-align: right;
}
.strength-text.weak   { color: #ef4444; }
.strength-text.fair   { color: #f59e0b; }
.strength-text.good   { color: #3b82f6; }
.strength-text.strong { color: #22c55e; }

/* Hints */
.field-hint {
  font-size: 0.78rem;
  font-weight: 500;
  margin-top: 2px;
}
.field-hint.error { color: #f87171; }
.field-hint.ok    { color: #4ade80; }

/* Error/Success messages */
.error-msg, .success-msg {
  border-radius: 10px;
  padding: 11px 14px;
  font-size: 0.875rem;
  display: flex;
  align-items: center;
  gap: 8px;
}

.error-msg {
  background: rgba(220, 38, 38, 0.1);
  border: 1px solid rgba(220, 38, 38, 0.3);
  color: #fca5a5;
}

.success-msg {
  background: rgba(34, 197, 94, 0.1);
  border: 1px solid rgba(34, 197, 94, 0.3);
  color: #4ade80;
}

/* Submit */
.btn-submit {
  width: 100%;
  padding: 13px;
  background: linear-gradient(135deg, #2563eb 0%, #7c3aed 100%);
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
  box-shadow: 0 4px 20px rgba(37, 99, 235, 0.35);
  margin-top: 4px;
}

.btn-submit:hover:not(:disabled) {
  opacity: 0.9;
  transform: translateY(-1px);
  box-shadow: 0 8px 28px rgba(37, 99, 235, 0.45);
}

.btn-submit:disabled { opacity: 0.5; cursor: not-allowed; }

/* Spinner */
.spinner {
  width: 17px;
  height: 17px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  flex-shrink: 0;
}
@keyframes spin { to { transform: rotate(360deg); } }

/* Switch link */
.switch-link {
  text-align: center;
  font-size: 0.82rem;
  color: #334155;
  margin: 20px 0 0;
}

.link-btn {
  background: none;
  border: none;
  color: #60a5fa;
  font-size: inherit;
  font-family: 'Inter', sans-serif;
  font-weight: 600;
  cursor: pointer;
  padding: 0 2px;
  transition: color 0.2s;
}
.link-btn:hover { color: #93c5fd; }

/* Fade */
.fade-enter-active, .fade-leave-active { transition: opacity 0.2s, transform 0.2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-4px); }

/* Responsive */
@media (max-width: 480px) {
  .register-card { padding: 28px 20px 24px; }
  .field-row { grid-template-columns: 1fr; }
  .brand-title { font-size: 1.35rem; }
}
</style>
