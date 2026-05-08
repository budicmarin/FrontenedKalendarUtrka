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

const props = defineProps<{ userId?: string }>()

const races = ref<Race[]>([])
const isLoading = ref(false)
const error = ref('')
const searchQuery = ref('')
const selectedFilter = ref<'all' | 'upcoming' | 'live' | 'completed' | 'myraces'>('all')
const selectedRace = ref<Race | null>(null)
const registeringId = ref<string | null>(null)

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

watch(() => props.userId, () => fetchRaces())
onMounted(() => fetchRaces())

async function toggleRegistration(race: Race, event: Event) {
  event.stopPropagation()
  if (!props.userId || registeringId.value) return
  registeringId.value = race._id
  const wasRegistered = race.isRegistered
  const target = races.value.find(r => r._id === race._id)
  if (target) target.isRegistered = !wasRegistered
  if (selectedRace.value?._id === race._id)
    selectedRace.value = { ...selectedRace.value, isRegistered: !wasRegistered }
  try {
    const method = wasRegistered ? 'DELETE' : 'POST'
    const res = await fetch(
      `https://backendkalendarutrka.onrender.com/api/raceuser/race/${race._id}/register`,
      { method, headers: { 'Content-Type': 'application/json' }, body: JSON.stringify({ userId: props.userId }) }
    )
    if (!res.ok) throw new Error('Registracija nije uspjela')
  } catch {
    if (target) target.isRegistered = wasRegistered
    if (selectedRace.value?._id === race._id)
      selectedRace.value = { ...selectedRace.value, isRegistered: wasRegistered }
  } finally {
    registeringId.value = null
  }
}

function getCalculatedStatus(dateStr: string): 'upcoming' | 'live' | 'completed' {
  const today = new Date(); today.setHours(0, 0, 0, 0)
  const raceDate = new Date(dateStr); raceDate.setHours(0, 0, 0, 0)
  if (raceDate.getTime() < today.getTime()) return 'completed'
  if (raceDate.getTime() === today.getTime()) return 'live'
  return 'upcoming'
}

const processedRaces = computed(() =>
  races.value.map(race => ({ ...race, status: getCalculatedStatus(race.datum) }))
)

const filteredRaces = computed(() =>
  processedRaces.value.filter(race => {
    if (selectedFilter.value === 'myraces') return race.isRegistered
    const matchesFilter = selectedFilter.value === 'all' || race.status === selectedFilter.value
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
  return new Date(dateStr).toLocaleDateString('hr-HR', { day: '2-digit', month: 'short', year: 'numeric' })
}

function getDaysUntil(dateStr: string): number {
  return Math.ceil((new Date(dateStr).getTime() - new Date().getTime()) / (1000 * 60 * 60 * 24))
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

    <!-- HERO -->
    <div class="hero">
      <div class="hero-glow"></div>
      <div class="hero-content">
        <p class="hero-eyebrow">🏃 Istra &amp; okolica</p>
        <h1 class="hero-title">Kalendar trkačkih utrka</h1>
        <p class="hero-sub">Sve nadolazeće i završene utrke na jednom mjestu</p>

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

    <!-- CONTROLS -->
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
        >{{ opt.label }}</button>
      </div>
    </div>

    <!-- LOADING -->
    <div v-if="isLoading" class="state-box">
      <div class="spinner-large"></div>
      <p>Učitavanje utrka...</p>
    </div>

    <!-- ERROR -->
    <div v-else-if="error" class="state-box">
      <p class="state-icon">⚠️</p>
      <p>{{ error }}</p>
      <button @click="fetchRaces" class="filter-btn retry-btn">Pokušaj ponovo</button>
    </div>

    <!-- GRID -->
    <div v-else class="races-grid">
      <div
        v-for="race in filteredRaces"
        :key="race._id"
        :class="['race-card', `status-${race.status ?? 'upcoming'}`]"
        @click="openModal(race)"
      >
        <div class="card-top">
          <div :class="['status-dot', race.status ?? 'upcoming']"></div>
          <h3 class="race-name">{{ race.naziv }}</h3>
        </div>

        <div class="card-meta">
          <span class="meta-row">📍 {{ race.lokacija }}</span>
          <span class="meta-row">📅 {{ formatDate(race.datum) }} &nbsp;🕐 {{ race.vrijeme }}</span>
        </div>

        <div class="card-footer">
          <span v-if="race.status === 'completed'" class="badge badge-completed">✅ Završena</span>
          <span v-else-if="race.status === 'live'" class="badge badge-live">🔴 Uživo</span>
          <span v-else class="badge badge-upcoming">
            <template v-if="getDaysUntil(race.datum) > 0">⏳ {{ getDaysUntil(race.datum) }} dana</template>
            <template v-else>🏁 Danas!</template>
          </span>

          <button
            v-if="userId"
            :class="['reg-btn', { registered: race.isRegistered }]"
            :disabled="registeringId === race._id"
            @click="toggleRegistration(race, $event)"
          >
            <span v-if="registeringId === race._id" class="btn-spinner"></span>
            <span v-else-if="race.isRegistered">✓ Prijavljen</span>
            <span v-else>+ Prijavi se</span>
          </button>
        </div>
      </div>

      <div v-if="filteredRaces.length === 0" class="empty-state">
        <div class="empty-icon">🏃</div>
        <p>Nema utrka koje odgovaraju pretrazi.</p>
      </div>
    </div>

    <!-- MODAL -->
    <Transition name="modal">
      <div v-if="selectedRace" class="modal-overlay" @click.self="closeModal">
        <div class="modal-card">
          <button class="modal-close" @click="closeModal">✕</button>

          <h2 class="modal-title">{{ selectedRace.naziv }}</h2>
          <p class="modal-loc">📍 {{ selectedRace.lokacija }}</p>
          <div class="modal-divider"></div>

          <div class="modal-info-grid">
            <div class="info-item">
              <span class="info-label">Datum</span>
              <span class="info-val">{{ formatDate(selectedRace.datum) }}</span>
            </div>
            <div class="info-item">
              <span class="info-label">Vrijeme</span>
              <span class="info-val">{{ selectedRace.vrijeme }}</span>
            </div>
            <div class="info-item">
              <span class="info-label">Lokacija</span>
              <span class="info-val">{{ selectedRace.lokacija }}</span>
            </div>
            <div class="info-item">
              <span class="info-label">Status</span>
              <span class="info-val">
                <span v-if="selectedRace.status === 'completed'" class="badge badge-completed">✅ Završena</span>
                <span v-else-if="selectedRace.status === 'live'" class="badge badge-live">🔴 Uživo</span>
                <span v-else class="badge badge-upcoming">⏳ {{ getDaysUntil(selectedRace.datum) > 0 ? getDaysUntil(selectedRace.datum) + ' dana' : 'Danas!' }}</span>
              </span>
            </div>
          </div>

          <button
            v-if="userId"
            :class="['modal-reg-btn', { registered: selectedRace.isRegistered }]"
            :disabled="registeringId === selectedRace._id"
            @click="toggleRegistration(selectedRace, $event)"
          >
            <span v-if="registeringId === selectedRace._id" class="btn-spinner modal-spinner"></span>
            <span v-else-if="selectedRace.isRegistered">✕ Odjavi se s utrke</span>
            <span v-else>⚡ Prijavi se na utrku</span>
          </button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;700;800&family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* ── BASE ── */
.calendar-page {
  min-height: 100vh;
  background: #080b14;
  font-family: 'DM Sans', sans-serif;
  color: #e2e8f0;
  padding-bottom: 80px;
}

/* ── HERO ── */
.hero {
  position: relative;
  text-align: center;
  padding: 80px 24px 64px;
  overflow: hidden;
}

.hero-glow {
  position: absolute;
  top: -120px; left: 50%;
  transform: translateX(-50%);
  width: min(800px, 120vw);
  height: 480px;
  background: radial-gradient(ellipse, rgba(220,38,38,.2) 0%, rgba(124,58,237,.1) 45%, transparent 70%);
  pointer-events: none;
}

.hero-content {
  position: relative;
  z-index: 1;
  max-width: 860px;
  margin: 0 auto;
}

.hero-eyebrow {
  font-size: .75rem;
  font-weight: 600;
  letter-spacing: .12em;
  text-transform: uppercase;
  color: #f87171;
  margin-bottom: 14px;
}

.hero-title {
  font-family: 'Syne', sans-serif;
  font-size: clamp(2rem, 6vw, 4rem);
  font-weight: 800;
  line-height: 1.1;
  background: linear-gradient(135deg, #fff 0%, #f87171 50%, #c026d3 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 14px;
}

.hero-sub {
  font-size: clamp(.85rem, 2vw, 1rem);
  color: #64748b;
  font-weight: 300;
  margin-bottom: 48px;
}

/* ── STATS ── */
.stats-row {
  display: flex;
  justify-content: center;
  gap: 12px;
  flex-wrap: wrap;
}

.stat-card {
  background: rgba(255,255,255,.04);
  border: 1px solid rgba(255,255,255,.08);
  border-radius: 16px;
  padding: 18px clamp(16px, 4vw, 32px);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  backdrop-filter: blur(8px);
  transition: transform .2s, border-color .2s;
  min-width: 90px;
}

.stat-card:hover { transform: translateY(-2px); border-color: rgba(255,255,255,.15); }

.stat-num {
  font-family: 'Syne', sans-serif;
  font-size: clamp(1.6rem, 4vw, 2.2rem);
  font-weight: 800;
  color: #fff;
  line-height: 1;
}

.stat-label {
  font-size: .68rem;
  text-transform: uppercase;
  letter-spacing: .1em;
  color: #475569;
  font-weight: 500;
}

.accent-green  { color: #22c55e !important; }
.accent-blue   { color: #60a5fa !important; }
.accent-red    { color: #f87171 !important; }
.accent-purple { color: #c084fc !important; }

.pulse { animation: pulse-anim 1.4s ease-in-out infinite; }
@keyframes pulse-anim { 0%,100%{opacity:1} 50%{opacity:.45} }

/* ── CONTROLS ── */
.controls {
  max-width: 1140px;
  margin: 0 auto 36px;
  padding: 0 clamp(16px, 4vw, 32px);
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.search-wrap {
  position: relative;
  width: 100%;
}

.search-icon {
  position: absolute;
  left: 16px;
  top: 50%;
  transform: translateY(-50%);
  font-size: .95rem;
  pointer-events: none;
}

.search-input {
  width: 100%;
  background: rgba(255,255,255,.05);
  border: 1px solid rgba(255,255,255,.1);
  border-radius: 14px;
  padding: 14px 18px 14px 46px;
  color: #e2e8f0;
  font-size: .9rem;
  font-family: 'DM Sans', sans-serif;
  outline: none;
  transition: border-color .2s, background .2s;
}

.search-input::placeholder { color: #334155; }
.search-input:focus {
  border-color: rgba(220,38,38,.5);
  background: rgba(255,255,255,.07);
}

.filter-tabs {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.filter-btn {
  background: rgba(255,255,255,.04);
  border: 1px solid rgba(255,255,255,.1);
  color: #64748b;
  padding: 9px 18px;
  border-radius: 999px;
  font-size: .82rem;
  font-weight: 500;
  cursor: pointer;
  font-family: 'DM Sans', sans-serif;
  transition: all .2s;
  white-space: nowrap;
}

.filter-btn:hover { border-color: rgba(255,255,255,.2); color: #e2e8f0; }
.filter-btn.active {
  background: linear-gradient(135deg, rgba(220,38,38,.2), rgba(192,38,211,.15));
  border-color: rgba(220,38,38,.5);
  color: #f87171;
}

/* ── GRID ── */
.races-grid {
  max-width: 1140px;
  margin: 0 auto;
  padding: 0 clamp(16px, 4vw, 32px);
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(min(300px, 100%), 1fr));
  gap: 16px;
}

/* ── CARD ── */
.race-card {
  position: relative;
  background: rgba(15,23,42,.85);
  border: 1px solid rgba(255,255,255,.08);
  border-radius: 20px;
  padding: 22px 20px 18px;
  cursor: pointer;
  transition: transform .25s, box-shadow .25s, border-color .25s;
  backdrop-filter: blur(10px);
  overflow: hidden;
}

.race-card::after {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 2px;
  background: linear-gradient(90deg, #dc2626, #c026d3);
  opacity: 0;
  transition: opacity .25s;
}

.race-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 16px 48px rgba(0,0,0,.45);
  border-color: rgba(220,38,38,.25);
}

.race-card:hover::after { opacity: 1; }
.race-card.status-completed { border-color: rgba(34,197,94,.12); }
.race-card.status-live { border-color: rgba(239,68,68,.35); animation: live-glow 2s ease-in-out infinite; }

@keyframes live-glow {
  0%,100% { box-shadow: 0 0 16px rgba(239,68,68,.08); }
  50%      { box-shadow: 0 0 36px rgba(239,68,68,.25); }
}

.card-top {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  margin-bottom: 12px;
}

.status-dot {
  width: 9px; height: 9px;
  border-radius: 50%;
  flex-shrink: 0;
  margin-top: 5px;
}

.status-dot.completed { background: #22c55e; }
.status-dot.upcoming  { background: #60a5fa; box-shadow: 0 0 7px rgba(96,165,250,.5); }
.status-dot.live      { background: #ef4444; animation: dot-pulse 1.2s ease-in-out infinite; }

@keyframes dot-pulse {
  0%,100% { box-shadow: 0 0 4px rgba(239,68,68,.4); }
  50%      { box-shadow: 0 0 12px rgba(239,68,68,.8); }
}

.race-name {
  font-family: 'Syne', sans-serif;
  font-size: clamp(.95rem, 2.5vw, 1.1rem);
  font-weight: 700;
  color: #f1f5f9;
  line-height: 1.3;
}

.card-meta {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 16px;
}

.meta-row {
  font-size: .82rem;
  color: #64748b;
  display: block;
}

.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding-top: 14px;
  border-top: 1px solid rgba(255,255,255,.06);
  flex-wrap: wrap;
}

/* ── BADGES ── */
.badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: .72rem;
  font-weight: 600;
  padding: 4px 10px;
  border-radius: 999px;
  letter-spacing: .02em;
  white-space: nowrap;
}

.badge-completed { background: rgba(34,197,94,.12);  color: #4ade80; border: 1px solid rgba(34,197,94,.2); }
.badge-upcoming  { background: rgba(96,165,250,.12);  color: #93c5fd; border: 1px solid rgba(96,165,250,.2); }
.badge-live      { background: rgba(239,68,68,.14);   color: #fca5a5; border: 1px solid rgba(239,68,68,.3); }

/* ── REG BTN (card) ── */
.reg-btn {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 6px 14px;
  border-radius: 999px;
  border: 1px solid rgba(96,165,250,.4);
  background: rgba(96,165,250,.08);
  color: #93c5fd;
  font-size: .75rem;
  font-weight: 600;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  transition: all .2s;
  white-space: nowrap;
}

.reg-btn:hover:not(:disabled) {
  background: rgba(96,165,250,.18);
  border-color: #60a5fa;
}

.reg-btn.registered {
  border-color: rgba(34,197,94,.4);
  background: rgba(34,197,94,.08);
  color: #4ade80;
}

.reg-btn.registered:hover:not(:disabled) {
  background: rgba(239,68,68,.1);
  border-color: rgba(239,68,68,.5);
  color: #fca5a5;
}

.reg-btn:disabled { opacity: .55; cursor: not-allowed; }

/* ── SPINNERS ── */
.btn-spinner {
  display: inline-block;
  width: 13px; height: 13px;
  border: 2px solid rgba(255,255,255,.25);
  border-top-color: currentColor;
  border-radius: 50%;
  animation: spin .7s linear infinite;
}

.modal-spinner { width: 18px; height: 18px; }
@keyframes spin { to { transform: rotate(360deg); } }

/* ── EMPTY / LOADING / ERROR ── */
.state-box {
  text-align: center;
  padding: 80px 24px;
  color: #475569;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.state-icon { font-size: 2rem; }

.spinner-large {
  width: 44px; height: 44px;
  border: 3px solid rgba(255,255,255,.08);
  border-top-color: #f87171;
  border-radius: 50%;
  animation: spin .8s linear infinite;
  margin-bottom: 8px;
}

.retry-btn { margin-top: 4px; }

.empty-state {
  grid-column: 1 / -1;
  text-align: center;
  padding: 80px 20px;
  color: #475569;
}

.empty-icon { font-size: 3rem; margin-bottom: 12px; }

/* ── MODAL OVERLAY ── */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.72);
  backdrop-filter: blur(8px);
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: clamp(12px, 4vw, 24px);
}

.modal-card {
  background: #0d1526;
  border: 1px solid rgba(255,255,255,.11);
  border-radius: 24px;
  padding: clamp(24px, 5vw, 40px);
  max-width: 520px;
  width: 100%;
  max-height: 90dvh;
  overflow-y: auto;
  position: relative;
  box-shadow: 0 40px 100px rgba(0,0,0,.65);
  scrollbar-width: thin;
  scrollbar-color: rgba(255,255,255,.1) transparent;
}

.modal-close {
  position: absolute;
  top: 18px; right: 18px;
  background: rgba(255,255,255,.06);
  border: 1px solid rgba(255,255,255,.1);
  color: #64748b;
  width: 34px; height: 34px;
  border-radius: 50%;
  cursor: pointer;
  font-size: .9rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all .2s;
}

.modal-close:hover { background: rgba(220,38,38,.2); color: #f87171; }

.modal-title {
  font-family: 'Syne', sans-serif;
  font-size: clamp(1.3rem, 4vw, 1.7rem);
  font-weight: 800;
  color: #f1f5f9;
  margin-bottom: 6px;
  padding-right: 40px;
}

.modal-loc {
  color: #64748b;
  font-size: .87rem;
  margin-bottom: 24px;
}

.modal-divider {
  height: 1px;
  background: rgba(255,255,255,.07);
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
  flex-direction: column;
  gap: 4px;
}

.info-label {
  font-size: .68rem;
  text-transform: uppercase;
  letter-spacing: .1em;
  color: #475569;
  font-weight: 600;
}

.info-val {
  font-size: .9rem;
  font-weight: 500;
  color: #cbd5e1;
}

/* ── MODAL REG BTN ── */
.modal-reg-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  width: 100%;
  padding: 15px 24px;
  border-radius: 14px;
  border: none;
  font-size: .95rem;
  font-weight: 700;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  transition: all .3s;
  background: linear-gradient(135deg, #3b82f6, #6366f1);
  color: #fff;
  box-shadow: 0 4px 24px rgba(99,102,241,.35);
}

.modal-reg-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 32px rgba(99,102,241,.5);
}

.modal-reg-btn.registered {
  background: linear-gradient(135deg, rgba(239,68,68,.2), rgba(185,28,28,.15));
  border: 1px solid rgba(239,68,68,.4);
  color: #fca5a5;
  box-shadow: 0 4px 20px rgba(239,68,68,.2);
}

.modal-reg-btn.registered:hover:not(:disabled) {
  box-shadow: 0 8px 28px rgba(239,68,68,.35);
}

.modal-reg-btn:disabled { opacity: .6; cursor: not-allowed; transform: none; }

/* ── TRANSITION ── */
.modal-enter-active, .modal-leave-active { transition: opacity .25s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; }
.modal-enter-active .modal-card, .modal-leave-active .modal-card { transition: transform .25s ease; }
.modal-enter-from .modal-card { transform: scale(.93) translateY(16px); }
.modal-leave-to .modal-card   { transform: scale(.93) translateY(16px); }

/* ── RESPONSIVE ── */
@media (max-width: 600px) {
  .hero { padding: 60px 16px 48px; }
  .stats-row { gap: 8px; }
  .stat-card { padding: 14px 16px; min-width: 80px; }
  .controls { padding: 0 16px; }
  .races-grid { padding: 0 16px; gap: 12px; }
  .filter-tabs { gap: 6px; }
  .filter-btn { padding: 8px 14px; font-size: .78rem; }
  .modal-info-grid { grid-template-columns: 1fr; }
  .card-footer { gap: 6px; }
}

@media (max-width: 380px) {
  .hero-title { font-size: 1.7rem; }
  .race-card { padding: 18px 16px 14px; }
}
</style>