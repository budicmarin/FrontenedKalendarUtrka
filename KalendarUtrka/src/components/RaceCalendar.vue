<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'

interface Race {
  _id: string
  naziv: string
  datum: string
  vrijeme: string
  lokacija: string
  status?: 'upcoming' | 'live' | 'completed'
  isRegistered?: boolean
}

// Prima userId od App.vue
const props = defineProps<{ userId?: string }>()

const races = ref<Race[]>([])
const isLoading = ref(false)
const error = ref('')
const searchQuery = ref('')
const selectedFilter = ref<'all' | 'upcoming' | 'live' | 'completed' | 'myraces'>('all')
const selectedRace = ref<Race | null>(null)
const registeringId = ref<string | null>(null)

// ── Dohvat utrka + prijava korisnika ──────────────────────────
async function fetchRaces() {
  isLoading.value = true
  error.value = ''
  try {
    const [racesRes, userRacesRes] = await Promise.all([
      fetch('https://backendkalendarutrka.onrender.com/api/races'),
      props.userId
        ? fetch(`https://backendkalendarutrka.onrender.com/api/raceuser/user/${props.userId}`)
        : Promise.resolve(null),
    ])

    if (!racesRes.ok) throw new Error('Failed to fetch races')
    const racesData: Race[] = await racesRes.json()

    let registeredRaceIds = new Set<string>()
    if (userRacesRes?.ok) {
      const userRaces = await userRacesRes.json()
      registeredRaceIds = new Set(userRaces.map((r: any) => r.race_id))
    }

    races.value = racesData.map(race => ({
      ...race,
      isRegistered: registeredRaceIds.has(race._id.toString()),
    }))
  } catch (err) {
    error.value = 'Nije moguće dohvatiti utrke.'
  } finally {
    isLoading.value = false
  }
}

// Ponovo dohvati kad se korisnik prijavi/odjavi
watch(() => props.userId, () => fetchRaces())
onMounted(() => fetchRaces())

// ── Prijava / odjava s utrke ──────────────────────────────────
async function toggleRegistration(race: Race, event: Event) {
  event.stopPropagation()
  if (!props.userId || registeringId.value) return

  registeringId.value = race._id
  const wasRegistered = race.isRegistered

  // Optimistično ažuriranje
  const target = races.value.find(r => r._id === race._id)
  if (target) target.isRegistered = !wasRegistered
  if (selectedRace.value?._id === race._id) {
    selectedRace.value = { ...selectedRace.value, isRegistered: !wasRegistered }
  }

  try {
    const method = wasRegistered ? 'DELETE' : 'POST'
    const res = await fetch(
      `http://localhost:3000/api/raceuser/race/${race._id}/register`,
      {
        method,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ userId: props.userId }),
      }
    )
    if (!res.ok) throw new Error('Registracija nije uspjela')
  } catch {
    // Rollback na grešku
    if (target) target.isRegistered = wasRegistered
    if (selectedRace.value?._id === race._id) {
      selectedRace.value = { ...selectedRace.value, isRegistered: wasRegistered }
    }
  } finally {
    registeringId.value = null
  }
}

// ── Status računanje ─────────────────────────────────────────
function getCalculatedStatus(dateStr: string): 'upcoming' | 'live' | 'completed' {
  const today = new Date()
  today.setHours(0, 0, 0, 0)
  const raceDate = new Date(dateStr)
  raceDate.setHours(0, 0, 0, 0)
  if (raceDate.getTime() < today.getTime()) return 'completed'
  if (raceDate.getTime() === today.getTime()) return 'live'
  return 'upcoming'
}

const processedRaces = computed(() =>
  races.value.map(race => ({ ...race, status: getCalculatedStatus(race.datum) }))
)

// ── Filtriranje ──────────────────────────────────────────────
const filteredRaces = computed(() =>
  processedRaces.value.filter(race => {
    if (selectedFilter.value === 'myraces') return race.isRegistered
    const matchesFilter =
      selectedFilter.value === 'all' || race.status === selectedFilter.value
    const matchesSearch =
      race.naziv.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      race.lokacija.toLowerCase().includes(searchQuery.value.toLowerCase())
    return matchesFilter && matchesSearch
  })
)

const stats = computed(() => ({
  total: processedRaces.value.length,
  completed: processedRaces.value.filter(r => r.status === 'completed').length,
  upcoming: processedRaces.value.filter(r => r.status === 'upcoming').length,
  live: processedRaces.value.filter(r => r.status === 'live').length,
  myraces: processedRaces.value.filter(r => r.isRegistered).length,
}))

function formatDate(dateStr: string): string {
  return new Date(dateStr).toLocaleDateString('hr-HR', {
    day: '2-digit', month: 'short', year: 'numeric',
  })
}

function getDaysUntil(dateStr: string): number {
  const diff = new Date(dateStr).getTime() - new Date().getTime()
  return Math.ceil(diff / (1000 * 60 * 60 * 24))
}

function openModal(race: Race) { selectedRace.value = race }
function closeModal() { selectedRace.value = null }

const filterOptions = [
  { key: 'all', label: 'Sve' },
  { key: 'upcoming', label: 'Nadolazeće' },
  { key: 'completed', label: 'Završene' },
  { key: 'live', label: 'Uživo' },
  { key: 'myraces', label: 'Moje utrke' },
] as const
</script>
<template>
  <div class="calendar-page">
    <div class="hero">
      <div class="hero-glow"></div>
      <div class="hero-content">
        <h1 class="hero-title">Kalendar trkačkih utrka</h1>
        <p class="hero-subtitle">Kalendar trenutnih utrka u trčanju Istra</p>
        <div class="stats-row">
          <div class="stat-card">
            <span class="stat-num">{{ stats.total }}</span>
            <span class="stat-label">Ukupno</span>
          </div>
          <div class="stat-card">
            <span class="stat-num accent-green">{{ stats.completed }}</span>
            <span class="stat-label">Završene</span>
          </div>
          <div class="stat-card">
            <span class="stat-num accent-blue">{{ stats.upcoming }}</span>
            <span class="stat-label">Nadolazeće</span>
          </div>
          <div class="stat-card" v-if="stats.live > 0">
            <span class="stat-num accent-red pulse">{{ stats.live }}</span>
            <span class="stat-label">Uživo</span>
          </div>
          <div class="stat-card" v-if="stats.myraces > 0">
            <span class="stat-num accent-purple">{{ stats.myraces }}</span>
            <span class="stat-label">Moje utrke</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <div class="search-wrap">
        <span class="search-icon">🔍</span>
        <input v-model="searchQuery" type="text" placeholder="Pretraži utrke..." class="search-input" />
      </div>
      <div class="filter-tabs">
        <button
          v-for="opt in filterOptions"
          :key="opt.key"
          :class="['filter-btn', { active: selectedFilter === opt.key }]"
          @click="selectedFilter = opt.key"
        >
          {{ opt.label }}
        </button>
      </div>
    </div>

    <!-- Loading / Error / Grid -->
    <div v-if="isLoading" class="loading-state">
      <div class="spinner-large"></div>
      <p>Učitavanje utrka...</p>
    </div>

    <div v-else-if="error" class="error-state">
      <p>⚠️ {{ error }}</p>
      <button @click="fetchRaces" class="filter-btn">Pokušaj ponovo</button>
    </div>

    <div v-else class="races-grid">
      <div
        v-for="race in filteredRaces"
        :key="race._id"
        :class="['race-card', race.status ? `status-${race.status}` : '']"
        @click="openModal(race)"
      >
        <!-- Status dot -->
        <div :class="['status-dot', race.status ?? 'upcoming']"></div>

        <!-- Naziv -->
        <h3 class="race-name">{{ race.naziv }}</h3>

        <!-- Lokacija -->
        <p class="race-location">📍 {{ race.lokacija }}</p>

        <!-- Datum i vrijeme -->
        <div class="race-date-row">
          <span class="race-date">📅 {{ formatDate(race.datum) }}</span>
          <span class="race-date"> 🕐 {{ race.vrijeme }}</span>
        </div>

        <!-- Status footer -->
        <div class="card-footer">
          <span v-if="race.status === 'completed'" class="status-label completed">✅ Završena</span>
          <span v-else-if="race.status === 'live'" class="status-label live">🔴 Uživo</span>
          <span v-else class="status-label upcoming">
            <template v-if="getDaysUntil(race.datum) > 0">⏳ {{ getDaysUntil(race.datum) }} dana</template>
            <template v-else>🏁 Danas!</template>
          </span>
          <button
  v-if="userId"
  :class="['register-btn', { registered: race.isRegistered }]"
  :disabled="registeringId === race._id"
  @click="toggleRegistration(race, $event)"
>
  <span v-if="registeringId === race._id">⏳</span>
  <span v-else-if="race.isRegistered">✅ Prijavljen</span>
  <span v-else>+ Prijavi se</span>
</button>
        </div>
      </div>

      <div v-if="filteredRaces.length === 0" class="empty-state">
        <div class="empty-icon">🏃</div>
        <p>Nema utrka koje odgovaraju pretrazi.</p>
      </div>
    </div>

    <!-- Modal -->
    <Transition name="modal">
      <div v-if="selectedRace" class="modal-overlay" @click.self="closeModal">
        <div class="modal-card">
          <button class="modal-close" @click="closeModal">✕</button>
          <div class="modal-header">
            <div>
              <h2 class="modal-title">{{ selectedRace.naziv }}</h2>
              <p class="modal-country">📍 {{ selectedRace.lokacija }}</p>
            </div>
          </div>
          <div class="modal-divider"></div>
          <div class="modal-info-grid">
            <div class="info-item">
              <span class="info-icon">📅</span>
              <div>
                <span class="info-label">Datum</span>
                <span class="info-value">{{ formatDate(selectedRace.datum) }}</span>
              </div>
            </div>
            <div class="info-item">
              <span class="info-icon">🕐</span>
              <div>
                <span class="info-label">Vrijeme</span>
                <span class="info-value">{{ selectedRace.vrijeme }}</span>
              </div>
            </div>
          </div>
          <div class="modal-status-banner" :class="selectedRace.status ?? 'upcoming'">
            <template v-if="selectedRace.status === 'completed'">✅ Utrka završena</template>
            <template v-else-if="selectedRace.status === 'live'">🔴 Utrka u tijeku</template>
            <template v-else>⏳ {{ getDaysUntil(selectedRace.datum) > 0 ? getDaysUntil(selectedRace.datum) + ' dana do utrke' : 'Danas!' }}</template>
          </div>
          <button
  v-if="userId"
  :class="['modal-register-btn', { registered: selectedRace?.isRegistered }]"
  :disabled="registeringId === selectedRace?._id"
  @click="selectedRace && toggleRegistration(selectedRace, $event)"
>
  <span v-if="registeringId === selectedRace?._id">⏳ Učitavanje...</span>
  <span v-else-if="selectedRace?.isRegistered">❌ Odjavi se s utrke</span>
  <span v-else>🏃 Prijavi se na utrku</span>
</button>
        </div>
      </div>
    </Transition>
  </div>
</template>
<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&family=Orbitron:wght@700;900&display=swap');

* {
  box-sizing: border-box;
}

.calendar-page {
  min-height: 100vh;
  background: #080b14;
  font-family: 'Inter', sans-serif;
  color: #e2e8f0;
  padding-bottom: 60px;
}

/* === HERO === */
.hero {
  position: relative;
  padding: 80px 24px 60px;
  text-align: center;
  overflow: hidden;
}

.hero-glow {
  position: absolute;
  top: -100px;
  left: 50%;
  transform: translateX(-50%);
  width: 700px;
  height: 400px;
  background: radial-gradient(ellipse, rgba(220, 38, 38, 0.25) 0%, rgba(124, 58, 237, 0.12) 50%, transparent 70%);
  pointer-events: none;
}

.hero-content {
  position: relative;
  z-index: 1;
  max-width: 800px;
  margin: 0 auto;
}

.season-badge {
  display: inline-block;
  background: rgba(220, 38, 38, 0.15);
  border: 1px solid rgba(220, 38, 38, 0.4);
  color: #f87171;
  font-size: 0.8rem;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  padding: 6px 18px;
  border-radius: 999px;
  margin-bottom: 20px;
}

.hero-title {
  font-family: 'Orbitron', sans-serif;
  font-size: clamp(2.4rem, 6vw, 4.5rem);
  font-weight: 900;
  background: linear-gradient(135deg, #ffffff 0%, #f87171 50%, #c026d3 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin: 0 0 16px;
  line-height: 1.1;
}

.hero-subtitle {
  color: #94a3b8;
  font-size: 1.05rem;
  margin: 0 0 40px;
  font-weight: 400;
}

/* === STATS === */
.stats-row {
  display: flex;
  justify-content: center;
  gap: 16px;
  flex-wrap: wrap;
}

.stat-card {
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 16px;
  padding: 20px 28px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  backdrop-filter: blur(8px);
  transition: transform 0.2s, border-color 0.2s;
}

.stat-card:hover {
  transform: translateY(-2px);
  border-color: rgba(255,255,255,0.15);
}

.stat-num {
  font-family: 'Orbitron', sans-serif;
  font-size: 2rem;
  font-weight: 700;
  color: #fff;
}

.stat-label {
  font-size: 0.75rem;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-weight: 500;
}

.accent-green { color: #22c55e !important; }
.accent-blue  { color: #60a5fa !important; }
.accent-red   { color: #f87171 !important; }
.accent-gold  { color: #fbbf24 !important; }
.accent-purple{ color: #c084fc !important; }

.pulse {
  animation: pulse 1.4s infinite;
}
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* === CONTROLS === */
.controls {
  max-width: 1200px;
  margin: 0 auto 40px;
  padding: 0 24px;
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  align-items: center;
}

.search-wrap {
  flex: 1;
  min-width: 220px;
  position: relative;
}

.search-icon {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 1rem;
  pointer-events: none;
}

.search-input {
  width: 100%;
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 12px;
  padding: 12px 16px 12px 44px;
  color: #e2e8f0;
  font-size: 0.95rem;
  font-family: 'Inter', sans-serif;
  outline: none;
  transition: border-color 0.2s, background 0.2s;
}

.search-input::placeholder { color: #475569; }
.search-input:focus {
  border-color: rgba(220, 38, 38, 0.5);
  background: rgba(255,255,255,0.07);
}

.filter-tabs {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.filter-btn {
  background: rgba(255,255,255,0.05);
  border: 1px solid rgba(255,255,255,0.1);
  color: #94a3b8;
  padding: 10px 20px;
  border-radius: 10px;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
  transition: all 0.2s;
}

.filter-btn:hover {
  background: rgba(255,255,255,0.09);
  color: #e2e8f0;
}

.filter-btn.active {
  background: rgba(220, 38, 38, 0.18);
  border-color: rgba(220, 38, 38, 0.5);
  color: #f87171;
}

/* === RACE GRID === */
.races-grid {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 24px;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.race-card {
  position: relative;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 20px;
  padding: 28px 24px 22px;
  cursor: pointer;
  transition: transform 0.25s, box-shadow 0.25s, border-color 0.25s;
  overflow: hidden;
}

.race-card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 3px;
  background: linear-gradient(90deg, #dc2626, #c026d3);
  opacity: 0;
  transition: opacity 0.25s;
}

.race-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 60px rgba(0,0,0,0.4);
  border-color: rgba(220, 38, 38, 0.3);
}

.race-card:hover::before {
  opacity: 1;
}

.race-card.status-completed { border-color: rgba(34, 197, 94, 0.15); }
.race-card.status-live { border-color: rgba(239, 68, 68, 0.4); animation: live-glow 2s infinite; }

@keyframes live-glow {
  0%, 100% { box-shadow: 0 0 20px rgba(239, 68, 68, 0.1); }
  50% { box-shadow: 0 0 40px rgba(239, 68, 68, 0.3); }
}

.round-badge {
  position: absolute;
  top: 18px;
  right: 18px;
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 8px;
  padding: 3px 10px;
  font-size: 0.72rem;
  font-weight: 600;
  color: #64748b;
  letter-spacing: 0.05em;
}

.status-dot {
  position: absolute;
  top: 22px;
  left: 24px;
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.status-dot.completed { background: #22c55e; }
.status-dot.upcoming  { background: #60a5fa; }
.status-dot.live      { background: #ef4444; animation: pulse 1.2s infinite; }

.race-flag {
  font-size: 2.8rem;
  margin-bottom: 8px;
  margin-top: 10px;
  line-height: 1;
}

.race-country {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: #64748b;
  margin-bottom: 10px;
}

.race-name {
  font-size: 1.05rem;
  font-weight: 700;
  color: #f1f5f9;
  margin: 0 0 6px;
  line-height: 1.3;
}

.race-location {
  font-size: 0.85rem;
  color: #64748b;
  margin: 0 0 14px;
}

.race-date-row {
  margin-bottom: 16px;
}

.race-date {
  font-size: 0.85rem;
  font-weight: 500;
  color: #94a3b8;
}

.card-footer {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
  border-top: 1px solid rgba(255,255,255,0.06);
  padding-top: 14px;
}

.status-label {
  font-size: 0.78rem;
  font-weight: 600;
  padding: 4px 12px;
  border-radius: 999px;
}

.status-label.completed {
  background: rgba(34, 197, 94, 0.12);
  color: #4ade80;
  border: 1px solid rgba(34, 197, 94, 0.25);
}

.status-label.upcoming {
  background: rgba(96, 165, 250, 0.12);
  color: #93c5fd;
  border: 1px solid rgba(96, 165, 250, 0.25);
}

.status-label.live {
  background: rgba(239, 68, 68, 0.15);
  color: #fca5a5;
  border: 1px solid rgba(239, 68, 68, 0.35);
}

.winner-chip {
  font-size: 0.75rem;
  font-weight: 600;
  color: #fbbf24;
  background: rgba(251, 191, 36, 0.1);
  border: 1px solid rgba(251, 191, 36, 0.2);
  padding: 3px 10px;
  border-radius: 999px;
  white-space: nowrap;
}

/* === EMPTY STATE === */
.empty-state {
  grid-column: 1 / -1;
  text-align: center;
  padding: 80px 20px;
  color: #475569;
}

.empty-icon {
  font-size: 3.5rem;
  margin-bottom: 16px;
}

/* === MODAL === */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.75);
  backdrop-filter: blur(6px);
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.modal-card {
  background: #0f1729;
  border: 1px solid rgba(255,255,255,0.1);
  border-radius: 24px;
  max-width: 520px;
  width: 100%;
  padding: 36px;
  position: relative;
  box-shadow: 0 40px 100px rgba(0,0,0,0.6);
}

.modal-close {
  position: absolute;
  top: 20px;
  right: 20px;
  background: rgba(255,255,255,0.07);
  border: 1px solid rgba(255,255,255,0.1);
  color: #94a3b8;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s, color 0.2s;
}

.modal-close:hover {
  background: rgba(220, 38, 38, 0.2);
  color: #f87171;
}

.modal-header {
  display: flex;
  align-items: flex-start;
  gap: 20px;
  margin-bottom: 24px;
}

.modal-flag {
  font-size: 3.5rem;
  flex-shrink: 0;
  line-height: 1;
}

.modal-round {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: #f87171;
  margin-bottom: 6px;
}

.modal-title {
  font-size: 1.4rem;
  font-weight: 700;
  color: #f1f5f9;
  margin: 0 0 4px;
  font-family: 'Inter', sans-serif;
}

.modal-country {
  font-size: 0.875rem;
  color: #64748b;
  margin: 0;
}

.modal-divider {
  height: 1px;
  background: rgba(255,255,255,0.07);
  margin-bottom: 24px;
}

.modal-info-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
  margin-bottom: 24px;
}

.info-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}

.info-icon {
  font-size: 1.2rem;
  flex-shrink: 0;
  margin-top: 2px;
}

.info-label {
  display: block;
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: #475569;
  margin-bottom: 3px;
  font-weight: 600;
}

.info-value {
  display: block;
  font-size: 0.9rem;
  font-weight: 600;
  color: #cbd5e1;
  line-height: 1.3;
}

.modal-status-banner {
  text-align: center;
  padding: 12px;
  border-radius: 12px;
  font-size: 0.9rem;
  font-weight: 600;
}

.modal-status-banner.completed {
  background: rgba(34, 197, 94, 0.1);
  color: #4ade80;
  border: 1px solid rgba(34, 197, 94, 0.2);
}

.modal-status-banner.upcoming {
  background: rgba(96, 165, 250, 0.1);
  color: #93c5fd;
  border: 1px solid rgba(96, 165, 250, 0.2);
}

.modal-status-banner.live {
  background: rgba(239, 68, 68, 0.15);
  color: #fca5a5;
  border: 1px solid rgba(239, 68, 68, 0.3);
}

/* === MODAL TRANSITION === */
.modal-enter-active, .modal-leave-active {
  transition: opacity 0.25s ease;
}
.modal-enter-from, .modal-leave-to {
  opacity: 0;
}
.modal-enter-active .modal-card,
.modal-leave-active .modal-card {
  transition: transform 0.25s ease;
}
.modal-enter-from .modal-card {
  transform: scale(0.92) translateY(20px);
}
.modal-leave-to .modal-card {
  transform: scale(0.92) translateY(20px);
}

/* === RESPONSIVE === */
@media (max-width: 640px) {
  .hero { padding: 60px 16px 40px; }
  .races-grid { padding: 0 16px; grid-template-columns: 1fr; }
  .controls { padding: 0 16px; }
  .modal-card { padding: 24px 20px; }
  .modal-info-grid { grid-template-columns: 1fr; }
  .stats-row { gap: 10px; }
  .stat-card { padding: 14px 20px; }
}
</style>
