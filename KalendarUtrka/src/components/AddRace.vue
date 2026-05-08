<script setup lang="ts">
import { ref } from 'vue'

const naziv = ref('')
const lokacija = ref('')
const datum = ref('')
const vrijeme = ref('')
const opis = ref('')
const isLoading = ref(false)
const errorMsg = ref('')
const successMsg = ref('')

const API_BASE = 'https://backendkalendarutrka.onrender.com'

async function addRace() {
  errorMsg.value = ''
  successMsg.value = ''

  if (!naziv.value || !lokacija.value || !datum.value || !vrijeme.value || !opis.value) {
    errorMsg.value = 'Sva polja su obavezna.'
    return
  }

  isLoading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/races`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        naziv: naziv.value,
        lokacija: lokacija.value,
        datum: datum.value,
        vrijeme: vrijeme.value,
        opis: opis.value,
      }),
    })
    if (!res.ok) throw new Error('Greška pri dodavanju.')
    successMsg.value = 'Utrka uspješno dodana! ✅'
    naziv.value = lokacija.value = datum.value = vrijeme.value = opis.value = ''
  } catch {
    errorMsg.value = 'Nije moguće dodati utrku. Pokušajte ponovo.'
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="add-race-page">
    <div class="bg-glow bg-glow--green"></div>
    <div class="bg-glow bg-glow--purple"></div>

    <div class="add-race-card">
      <div class="brand">
        <div class="brand-icon">➕</div>
        <h1 class="brand-title">Nova utrka</h1>
        <p class="brand-subtitle">Dodajte novu utrku u kalendar</p>
      </div>

      <form class="form" @submit.prevent="addRace" novalidate>

        <Transition name="fade">
          <div v-if="errorMsg" class="error-msg" role="alert"><span>⚠️</span> {{ errorMsg }}</div>
        </Transition>
        <Transition name="fade">
          <div v-if="successMsg" class="success-msg" role="status"><span>🎉</span> {{ successMsg }}</div>
        </Transition>

        <div class="field">
          <label class="field-label" for="naziv">Naziv utrke <span class="req">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">🏁</span>
            <input id="naziv" v-model="naziv" type="text" class="field-input"
              placeholder="npr. Pulska x-ica" :disabled="isLoading" />
          </div>
        </div>

        <div class="field">
          <label class="field-label" for="lokacija">Lokacija <span class="req">*</span></label>
          <div class="field-wrap">
            <span class="field-icon">📍</span>
            <input id="lokacija" v-model="lokacija" type="text" class="field-input"
              placeholder="Grad, država" :disabled="isLoading" />
          </div>
        </div>

        <div class="field-row">
          <div class="field">
            <label class="field-label" for="datum">Datum <span class="req">*</span></label>
            <div class="field-wrap">
              <span class="field-icon">📅</span>
              <input id="datum" v-model="datum" type="date" class="field-input date-input" :disabled="isLoading" />
            </div>
          </div>

          <div class="field">
            <label class="field-label" for="vrijeme">Vrijeme <span class="req">*</span></label>
            <div class="field-wrap">
              <span class="field-icon">⏰</span>
              <input id="vrijeme" v-model="vrijeme" type="time" class="field-input date-input" :disabled="isLoading" />
            </div>
          </div>
        </div>

        <div class="field">
          <label class="field-label" for="opis">Opis <span class="req">*</span></label>
          <div class="field-wrap textarea-wrap">
            <textarea
              id="opis"
              v-model="opis"
              class="field-input field-textarea"
              placeholder="Kratak opis utrke, trasa, kategorije..."
              rows="3"
              :disabled="isLoading"
            ></textarea>
          </div>
        </div>

        <button type="submit" class="btn-submit" :disabled="isLoading">
          <span v-if="isLoading" class="spinner"></span>
          <span>{{ isLoading ? 'Dodavanje...' : '+ Dodaj utrku' }}</span>
        </button>
      </form>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;700;800&family=DM+Sans:wght@300;400;500;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

.add-race-page {
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
  opacity: .25;
}

.bg-glow--green {
  width: min(500px, 90vw); height: min(500px, 90vw);
  background: radial-gradient(circle, #16a34a 0%, transparent 70%);
  top: -180px; right: -100px;
}

.bg-glow--purple {
  width: min(500px, 90vw); height: min(500px, 90vw);
  background: radial-gradient(circle, #7c3aed 0%, transparent 70%);
  bottom: -180px; left: -120px;
}

/* ── Card ── */
.add-race-card {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 520px;
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
  background: linear-gradient(135deg, #fff 0%, #4ade80 50%, #c026d3 100%);
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
  gap: 16px;
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

.req { color: #4ade80; }

.field-wrap {
  position: relative;
  display: flex;
  align-items: center;
}

.textarea-wrap { align-items: flex-start; }

.field-icon {
  position: absolute;
  left: 13px;
  top: 50%;
  transform: translateY(-50%);
  font-size: .9rem;
  pointer-events: none;
}

.textarea-wrap .field-icon {
  top: 14px;
  transform: none;
}

.field-input {
  width: 100%;
  background: rgba(255,255,255,.05);
  border: 1px solid rgba(255,255,255,.1);
  border-radius: 11px;
  padding: 12px 16px 12px 40px;
  color: #e2e8f0;
  font-size: .88rem;
  font-family: 'DM Sans', sans-serif;
  outline: none;
  transition: border-color .2s, background .2s, box-shadow .2s;
}

.field-input::placeholder { color: #334155; }

.field-input:focus {
  border-color: rgba(74,222,128,.45);
  background: rgba(255,255,255,.07);
  box-shadow: 0 0 0 3px rgba(74,222,128,.08);
}

.field-input:disabled { opacity: .5; cursor: not-allowed; }

/* Date/time inputs */
.date-input {
  color-scheme: dark;
}

/* Textarea */
.field-textarea {
  resize: vertical;
  min-height: 90px;
  line-height: 1.5;
  padding-top: 13px;
}

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
  padding: 14px;
  background: linear-gradient(135deg, #16a34a, #059669);
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
  box-shadow: 0 4px 20px rgba(22,163,74,.35);
  transition: opacity .2s, transform .15s, box-shadow .2s;
  margin-top: 4px;
}

.btn-submit:hover:not(:disabled) {
  opacity: .9;
  transform: translateY(-1px);
  box-shadow: 0 8px 28px rgba(22,163,74,.45);
}

.btn-submit:disabled { opacity: .5; cursor: not-allowed; }

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

/* ── Transition ── */
.fade-enter-active, .fade-leave-active { transition: opacity .2s, transform .2s; }
.fade-enter-from, .fade-leave-to { opacity: 0; transform: translateY(-5px); }

/* ── Responsive ── */
@media (max-width: 480px) {
  .field-row { grid-template-columns: 1fr; }
  .add-race-card { border-radius: 20px; }
  .field-input { font-size: .86rem; }
}
</style>