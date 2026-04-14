<script setup lang="ts">
import { ref, computed } from 'vue'

interface Race {
  id: number
  name: string
  location: string
  country: string
  flag: string
  date: string
  endDate: string
  circuit: string
  status: 'upcoming' | 'live' | 'completed'
  round: number
  winner?: string
  fastestLap?: string
  laps?: number
  distance?: string
  image?: string
}

const selectedFilter = ref<'all' | 'upcoming' | 'completed' | 'live'>('all')
const selectedRace = ref<Race | null>(null)
const searchQuery = ref('')

const races: Race[] = [
  {
    id: 1, round: 1, name: 'Bahrain Grand Prix', location: 'Sakhir', country: 'Bahrain', flag: '🇧🇭',
    date: '2025-03-02', endDate: '2025-03-02', circuit: 'Bahrain International Circuit',
    status: 'completed', winner: 'Max Verstappen', fastestLap: 'Lewis Hamilton',
    laps: 57, distance: '308.238 km'
  },
  {
    id: 2, round: 2, name: 'Saudi Arabian Grand Prix', location: 'Jeddah', country: 'Saudi Arabia', flag: '🇸🇦',
    date: '2025-03-09', endDate: '2025-03-09', circuit: 'Jeddah Corniche Circuit',
    status: 'completed', winner: 'Charles Leclerc', fastestLap: 'Oscar Piastri',
    laps: 50, distance: '308.450 km'
  },
  {
    id: 3, round: 3, name: 'Australian Grand Prix', location: 'Melbourne', country: 'Australia', flag: '🇦🇺',
    date: '2025-03-23', endDate: '2025-03-23', circuit: 'Albert Park Circuit',
    status: 'completed', winner: 'Lando Norris', fastestLap: 'Max Verstappen',
    laps: 58, distance: '307.574 km'
  },
  {
    id: 4, round: 4, name: 'Japanese Grand Prix', location: 'Suzuka', country: 'Japan', flag: '🇯🇵',
    date: '2025-04-06', endDate: '2025-04-06', circuit: 'Suzuka International Racing Course',
    status: 'completed', winner: 'Max Verstappen', fastestLap: 'Lando Norris',
    laps: 53, distance: '307.471 km'
  },
  {
    id: 5, round: 5, name: 'Chinese Grand Prix', location: 'Shanghai', country: 'China', flag: '🇨🇳',
    date: '2025-04-20', endDate: '2025-04-20', circuit: 'Shanghai International Circuit',
    status: 'upcoming', laps: 56, distance: '305.066 km'
  },
  {
    id: 6, round: 6, name: 'Miami Grand Prix', location: 'Miami', country: 'USA', flag: '🇺🇸',
    date: '2025-05-04', endDate: '2025-05-04', circuit: 'Miami International Autodrome',
    status: 'upcoming', laps: 57, distance: '308.326 km'
  },
  {
    id: 7, round: 7, name: 'Emilia Romagna Grand Prix', location: 'Imola', country: 'Italy', flag: '🇮🇹',
    date: '2025-05-18', endDate: '2025-05-18', circuit: 'Autodromo Enzo e Dino Ferrari',
    status: 'upcoming', laps: 63, distance: '309.049 km'
  },
  {
    id: 8, round: 8, name: 'Monaco Grand Prix', location: 'Monte Carlo', country: 'Monaco', flag: '🇲🇨',
    date: '2025-05-25', endDate: '2025-05-25', circuit: 'Circuit de Monaco',
    status: 'upcoming', laps: 78, distance: '260.286 km'
  },
  {
    id: 9, round: 9, name: 'Canadian Grand Prix', location: 'Montreal', country: 'Canada', flag: '🇨🇦',
    date: '2025-06-15', endDate: '2025-06-15', circuit: 'Circuit Gilles Villeneuve',
    status: 'upcoming', laps: 70, distance: '305.270 km'
  },
  {
    id: 10, round: 10, name: 'Spanish Grand Prix', location: 'Barcelona', country: 'Spain', flag: '🇪🇸',
    date: '2025-06-29', endDate: '2025-06-29', circuit: 'Circuit de Barcelona-Catalunya',
    status: 'upcoming', laps: 66, distance: '307.236 km'
  },
  {
    id: 11, round: 11, name: 'Austrian Grand Prix', location: 'Spielberg', country: 'Austria', flag: '🇦🇹',
    date: '2025-07-06', endDate: '2025-07-06', circuit: 'Red Bull Ring',
    status: 'upcoming', laps: 71, distance: '306.452 km'
  },
  {
    id: 12, round: 12, name: 'British Grand Prix', location: 'Silverstone', country: 'United Kingdom', flag: '🇬🇧',
    date: '2025-07-06', endDate: '2025-07-06', circuit: 'Silverstone Circuit',
    status: 'upcoming', laps: 52, distance: '306.198 km'
  },
  {
    id: 13, round: 13, name: 'Hungarian Grand Prix', location: 'Budapest', country: 'Hungary', flag: '🇭🇺',
    date: '2025-08-03', endDate: '2025-08-03', circuit: 'Hungaroring',
    status: 'upcoming', laps: 70, distance: '306.630 km'
  },
  {
    id: 14, round: 14, name: 'Belgian Grand Prix', location: 'Spa', country: 'Belgium', flag: '🇧🇪',
    date: '2025-08-24', endDate: '2025-08-24', circuit: 'Circuit de Spa-Francorchamps',
    status: 'upcoming', laps: 44, distance: '308.052 km'
  },
  {
    id: 15, round: 15, name: 'Dutch Grand Prix', location: 'Zandvoort', country: 'Netherlands', flag: '🇳🇱',
    date: '2025-08-31', endDate: '2025-08-31', circuit: 'Circuit Zandvoort',
    status: 'upcoming', laps: 72, distance: '306.587 km'
  },
  {
    id: 16, round: 16, name: 'Italian Grand Prix', location: 'Monza', country: 'Italy', flag: '🇮🇹',
    date: '2025-09-07', endDate: '2025-09-07', circuit: 'Autodromo Nazionale Monza',
    status: 'upcoming', laps: 53, distance: '306.720 km'
  },
  {
    id: 17, round: 17, name: 'Azerbaijan Grand Prix', location: 'Baku', country: 'Azerbaijan', flag: '🇦🇿',
    date: '2025-09-21', endDate: '2025-09-21', circuit: 'Baku City Circuit',
    status: 'upcoming', laps: 51, distance: '306.049 km'
  },
  {
    id: 18, round: 18, name: 'Singapore Grand Prix', location: 'Singapore', country: 'Singapore', flag: '🇸🇬',
    date: '2025-10-05', endDate: '2025-10-05', circuit: 'Marina Bay Street Circuit',
    status: 'upcoming', laps: 62, distance: '308.706 km'
  },
  {
    id: 19, round: 19, name: 'United States Grand Prix', location: 'Austin', country: 'USA', flag: '🇺🇸',
    date: '2025-10-19', endDate: '2025-10-19', circuit: 'Circuit of the Americas',
    status: 'upcoming', laps: 56, distance: '308.405 km'
  },
  {
    id: 20, round: 20, name: 'Mexico City Grand Prix', location: 'Mexico City', country: 'Mexico', flag: '🇲🇽',
    date: '2025-10-26', endDate: '2025-10-26', circuit: 'Autodromo Hermanos Rodriguez',
    status: 'upcoming', laps: 71, distance: '305.354 km'
  },
  {
    id: 21, round: 21, name: 'São Paulo Grand Prix', location: 'São Paulo', country: 'Brazil', flag: '🇧🇷',
    date: '2025-11-09', endDate: '2025-11-09', circuit: 'Autodromo Jose Carlos Pace',
    status: 'upcoming', laps: 71, distance: '305.879 km'
  },
  {
    id: 22, round: 22, name: 'Las Vegas Grand Prix', location: 'Las Vegas', country: 'USA', flag: '🇺🇸',
    date: '2025-11-22', endDate: '2025-11-22', circuit: 'Las Vegas Strip Circuit',
    status: 'upcoming', laps: 50, distance: '309.958 km'
  },
  {
    id: 23, round: 23, name: 'Qatar Grand Prix', location: 'Lusail', country: 'Qatar', flag: '🇶🇦',
    date: '2025-11-30', endDate: '2025-11-30', circuit: 'Lusail International Circuit',
    status: 'upcoming', laps: 57, distance: '308.611 km'
  },
  {
    id: 24, round: 24, name: 'Abu Dhabi Grand Prix', location: 'Abu Dhabi', country: 'UAE', flag: '🇦🇪',
    date: '2025-12-07', endDate: '2025-12-07', circuit: 'Yas Marina Circuit',
    status: 'upcoming', laps: 58, distance: '306.183 km'
  },
]

const filteredRaces = computed(() => {
  return races.filter(race => {
    const matchesFilter = selectedFilter.value === 'all' || race.status === selectedFilter.value
    const matchesSearch = race.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      race.location.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      race.country.toLowerCase().includes(searchQuery.value.toLowerCase())
    return matchesFilter && matchesSearch
  })
})

const stats = computed(() => ({
  total: races.length,
  completed: races.filter(r => r.status === 'completed').length,
  upcoming: races.filter(r => r.status === 'upcoming').length,
  live: races.filter(r => r.status === 'live').length,
}))

function formatDate(dateStr: string): string {
  const date = new Date(dateStr)
  return date.toLocaleDateString('en-GB', { day: '2-digit', month: 'short', year: 'numeric' })
}

function getDaysUntil(dateStr: string): number {
  const today = new Date()
  const raceDate = new Date(dateStr)
  const diff = raceDate.getTime() - today.getTime()
  return Math.ceil(diff / (1000 * 60 * 60 * 24))
}

function openModal(race: Race) {
  selectedRace.value = race
}

function closeModal() {
  selectedRace.value = null
}

const filterOptions = [
  { key: 'all', label: 'All Races' },
  { key: 'upcoming', label: 'Upcoming' },
  { key: 'completed', label: 'Completed' },
  { key: 'live', label: 'Live' },
] as const
</script>

<template>
  <div class="calendar-page">
    <!-- Header -->
    <div class="hero">
      <div class="hero-glow"></div>
      <div class="hero-content">
        <div class="season-badge"></div>
        <h1 class="hero-title">Kalendar trkačkih utrka</h1>
        <p class="hero-subtitle">Kalendar trenutnih utrka u trčanju Istra</p>

        <!-- Stats -->
        <div class="stats-row">
          <div class="stat-card">
            <span class="stat-num">{{ stats.total }}</span>
            <span class="stat-label">Total Races</span>
          </div>
          <div class="stat-card">
            <span class="stat-num accent-green">{{ stats.completed }}</span>
            <span class="stat-label">Completed</span>
          </div>
          <div class="stat-card">
            <span class="stat-num accent-blue">{{ stats.upcoming }}</span>
            <span class="stat-label">Upcoming</span>
          </div>
          <div class="stat-card" v-if="stats.live > 0">
            <span class="stat-num accent-red pulse">{{ stats.live }}</span>
            <span class="stat-label">Live Now</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <div class="search-wrap">
        <span class="search-icon">🔍</span>
        <input
          v-model="searchQuery"
          type="text"
          placeholder="Search races, locations..."
          class="search-input"
          id="race-search"
        />
      </div>
      <div class="filter-tabs">
        <button
          v-for="opt in filterOptions"
          :key="opt.key"
          :class="['filter-btn', { active: selectedFilter === opt.key }]"
          @click="selectedFilter = opt.key"
          :id="`filter-${opt.key}`"
        >
          {{ opt.label }}
        </button>
      </div>
    </div>

    <!-- Race Grid -->
    <div class="races-grid">
      <div
        v-for="race in filteredRaces"
        :key="race.id"
        :class="['race-card', `status-${race.status}`]"
        @click="openModal(race)"
        :id="`race-card-${race.id}`"
      >
        <!-- Round badge -->
        <div class="round-badge">R{{ race.round }}</div>

        <!-- Status indicator -->
        <div :class="['status-dot', race.status]"></div>

        <!-- Country & flag -->
        <div class="race-flag">{{ race.flag }}</div>
        <div class="race-country">{{ race.country }}</div>

        <!-- Race name -->
        <h3 class="race-name">{{ race.name }}</h3>
        <p class="race-location">📍 {{ race.location }}</p>

        <!-- Date -->
        <div class="race-date-row">
          <span class="race-date">📅 {{ formatDate(race.date) }}</span>
        </div>

        <!-- Status label -->
        <div class="card-footer">
          <span v-if="race.status === 'completed'" class="status-label completed">
            ✅ Completed
          </span>
          <span v-else-if="race.status === 'live'" class="status-label live">
            🔴 Live Now
          </span>
          <span v-else class="status-label upcoming">
            <template v-if="getDaysUntil(race.date) > 0">
              ⏳ {{ getDaysUntil(race.date) }} days
            </template>
            <template v-else>
              🏁 Today!
            </template>
          </span>

          <span v-if="race.status === 'completed' && race.winner" class="winner-chip">
            🏆 {{ race.winner }}
          </span>
        </div>
      </div>

      <!-- Empty state -->
      <div v-if="filteredRaces.length === 0" class="empty-state">
        <div class="empty-icon">🏎️</div>
        <p>No races found matching your search.</p>
      </div>
    </div>

    <!-- Modal -->
    <Transition name="modal">
      <div v-if="selectedRace" class="modal-overlay" @click.self="closeModal" id="race-modal">
        <div class="modal-card">
          <button class="modal-close" @click="closeModal" id="modal-close-btn">✕</button>

          <div class="modal-header">
            <span class="modal-flag">{{ selectedRace.flag }}</span>
            <div>
              <div class="modal-round">Round {{ selectedRace.round }}</div>
              <h2 class="modal-title">{{ selectedRace.name }}</h2>
              <p class="modal-country">{{ selectedRace.location }}, {{ selectedRace.country }}</p>
            </div>
          </div>

          <div class="modal-divider"></div>

          <div class="modal-info-grid">
            <div class="info-item">
              <span class="info-icon">📅</span>
              <div>
                <span class="info-label">Race Date</span>
                <span class="info-value">{{ formatDate(selectedRace.date) }}</span>
              </div>
            </div>
            <div class="info-item">
              <span class="info-icon">🏟️</span>
              <div>
                <span class="info-label">Circuit</span>
                <span class="info-value">{{ selectedRace.circuit }}</span>
              </div>
            </div>
            <div class="info-item" v-if="selectedRace.laps">
              <span class="info-icon">🔁</span>
              <div>
                <span class="info-label">Laps</span>
                <span class="info-value">{{ selectedRace.laps }}</span>
              </div>
            </div>
            <div class="info-item" v-if="selectedRace.distance">
              <span class="info-icon">📏</span>
              <div>
                <span class="info-label">Race Distance</span>
                <span class="info-value">{{ selectedRace.distance }}</span>
              </div>
            </div>
            <div class="info-item" v-if="selectedRace.winner">
              <span class="info-icon">🏆</span>
              <div>
                <span class="info-label">Winner</span>
                <span class="info-value accent-gold">{{ selectedRace.winner }}</span>
              </div>
            </div>
            <div class="info-item" v-if="selectedRace.fastestLap">
              <span class="info-icon">⚡</span>
              <div>
                <span class="info-label">Fastest Lap</span>
                <span class="info-value accent-purple">{{ selectedRace.fastestLap }}</span>
              </div>
            </div>
          </div>

          <div class="modal-status-banner" :class="selectedRace.status">
            <template v-if="selectedRace.status === 'completed'">✅ Race Completed</template>
            <template v-else-if="selectedRace.status === 'live'">🔴 Race in Progress</template>
            <template v-else>
              ⏳ {{ getDaysUntil(selectedRace.date) > 0 ? getDaysUntil(selectedRace.date) + ' days until race' : 'Race Today!' }}
            </template>
          </div>
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
