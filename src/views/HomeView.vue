<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import { useHotelStore } from '../store/hotelStore'
import HotelCard from '../components/HotelCard.vue'
import DateRangePicker from '../components/DateRangePicker.vue'
import GuestSelector from '../components/GuestSelector.vue'

const { state, getHotelsByCity } = useHotelStore()
const searchCity = ref('')
const showDatePicker = ref(false)
const showGuestPicker = ref(false)
const selectedDateRange = ref<{ start: Date | null, end: Date | null }>({ start: null, end: null })
const guestsState = ref({ adults: 2, children: 0, rooms: 1, pets: false })
const searchBarRef = ref<HTMLElement | null>(null)

const handleClickOutside = (event: MouseEvent) => {
  if (searchBarRef.value && !searchBarRef.value.contains(event.target as Node)) {
    showDatePicker.value = false
    showGuestPicker.value = false
  }
}

onMounted(() => {
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})

const toggleDatePicker = () => {
  showDatePicker.value = !showDatePicker.value
  if (showDatePicker.value) showGuestPicker.value = false
}

const toggleGuestPicker = () => {
  showGuestPicker.value = !showGuestPicker.value
  if (showGuestPicker.value) showDatePicker.value = false
}


const formatGuestsText = computed(() => {
  const { adults, children, rooms } = guestsState.value
  const people = adults + children
  return `${people} persona${people > 1 ? 's' : ''}, ${rooms} hab`
})

const formatDateRange = computed(() => {
  if (!selectedDateRange.value.start) return 'Agregar fechas'
  const options: Intl.DateTimeFormatOptions = { weekday: 'short', day: 'numeric', month: 'short' }
  const start = selectedDateRange.value.start.toLocaleDateString('es-ES', options).replace('.', '')
  if (!selectedDateRange.value.end) return `${start} — Salida`
  const end = selectedDateRange.value.end.toLocaleDateString('es-ES', options).replace('.', '')
  return `${start} — ${end}`
})

const filteredHotels = computed(() => {
  let result = state.hotels

  // 1. Filtrar por ciudad
  if (searchCity.value.trim()) {
    const lower = searchCity.value.toLowerCase()
    result = result.filter(h => h.city.toLowerCase().includes(lower))
  }

  // 2. Filtrar por capacidad (Huéspedes)
  const totalGuests = guestsState.value.adults + guestsState.value.children

  // 3. Evaluar disponibilidad de habitaciones
  result = result.filter(hotel => {
    const hotelRooms = state.rooms.filter(r => r.hotelId === hotel.id)
    
    // El hotel pasa el filtro si tiene AL MENOS UNA habitación disponible
    return hotelRooms.some(room => {
      // Verificación base: capacidad suficiente y no estar bloqueada manualmente
      let isRoomAvailable = room.isAvailable && room.capacity >= totalGuests

      // Verificación de fechas: si el usuario seleccionó un rango, validamos contra reservas existentes
      if (isRoomAvailable && selectedDateRange.value.start && selectedDateRange.value.end) {
        const searchStart = selectedDateRange.value.start.getTime()
        const searchEnd = selectedDateRange.value.end.getTime()
        
        const hasOverlap = state.reservations.some(res => {
          if (res.roomId !== room.id || res.status === 'Cancelled') return false
          const resStart = new Date(res.checkIn).getTime()
          const resEnd = new Date(res.checkOut).getTime()
          
          // Hay solapamiento si la fecha de entrada buscada es menor a la salida reservada
          // Y la fecha de salida buscada es mayor a la entrada reservada.
          return searchStart < resEnd && searchEnd > resStart
        })
        
        if (hasOverlap) isRoomAvailable = false
      }

      return isRoomAvailable
    })
  })

  return result
})
</script>

<template>
  <div class="view-container">
    <header class="view-header hero-section">
      <h1>Encuentra tu próximo destino</h1>
      <p>Descubre hoteles de lujo en las mejores ciudades</p>
      
      <div class="search-bar-complex animate-fade-in" ref="searchBarRef">
        <div class="search-field">
          <span class="field-icon">📍</span>
          <div class="field-content">
            <label>¿A dónde quieres ir?</label>
            <input 
              v-model="searchCity"
              type="text" 
              placeholder="Ej. Panamá"
              class="search-input"
            />
          </div>
        </div>
        
        <div class="field-divider"></div>
        
        <div class="search-field date-field" @click="toggleDatePicker">
          <span class="field-icon">📅</span>
          <div class="field-content">
            <label>Fechas</label>
            <div class="date-display">{{ formatDateRange }}</div>
          </div>
        </div>
        
        <!-- Popover Calendar -->
        <div v-if="showDatePicker" class="date-picker-popover">
          <DateRangePicker 
            v-model="selectedDateRange" 
            @close="showDatePicker = false"
          />
        </div>
        
        <div class="field-divider"></div>
        
        <div class="search-field" @click="toggleGuestPicker">
          <span class="field-icon">👤</span>
          <div class="field-content">
            <label>Huéspedes</label>
            <div class="date-display">{{ formatGuestsText }}</div>
          </div>
        </div>

        <!-- Popover Guests -->
        <div v-if="showGuestPicker" class="guest-picker-popover">
          <GuestSelector 
            v-model="guestsState" 
            @close="showGuestPicker = false"
          />
        </div>
        
        <button class="btn-search" @click="searchCity = searchCity.trim()">
          Buscar
        </button>
      </div>
    </header>
    
    <main class="view-content">
      <div class="results-header">
        <h2>{{ searchCity ? `Resultados para "${searchCity}"` : 'Hoteles Destacados' }}</h2>
        <span class="results-count">{{ filteredHotels.length }} hoteles encontrados</span>
      </div>

      <div v-if="filteredHotels.length > 0" class="hotels-grid">
        <HotelCard 
          v-for="hotel in filteredHotels" 
          :key="hotel.id" 
          :hotel="hotel" 
        />
      </div>
      
      <div v-else class="empty-state animate-fade-in">
        <div class="empty-icon">🏙️</div>
        <h3>No encontramos hoteles en esta ciudad</h3>
        <p>Intenta buscar otro destino o revisa nuestra lista de hoteles destacados borrando la búsqueda.</p>
        <button class="btn btn-primary" @click="searchCity = ''">Ver todos los hoteles</button>
      </div>
    </main>
  </div>
</template>

<style scoped>
.view-container {
  padding: 0; /* Hero section takes full width */
}

.hero-section {
  background: linear-gradient(135deg, var(--color-primary-dark) 0%, var(--color-primary) 100%);
  color: white;
  padding: 4rem 2rem;
  text-align: center;
  margin-bottom: var(--spacing-xl);
}

.hero-section h1 {
  font-size: var(--font-size-xxl);
  margin-bottom: var(--spacing-xs);
  letter-spacing: -0.02em;
}

.hero-section p {
  font-size: var(--font-size-lg);
  opacity: 0.9;
  margin-bottom: var(--spacing-xl);
}

.search-bar-complex {
  max-width: 1000px;
  margin: 0 auto;
  background: white;
  border-radius: 999px;
  display: flex;
  align-items: center;
  box-shadow: 0 4px 20px rgba(0,0,0,0.15);
  padding: 8px;
  color: var(--color-text);
  position: relative;
  z-index: 1000;
}

.date-picker-popover,
.guest-picker-popover {
  position: absolute;
  top: calc(100% + 12px);
  z-index: 100;
}

.date-picker-popover {
  left: 30%;
}

.guest-picker-popover {
  right: 15%;
}

.date-display {
  font-size: 0.95rem;
  color: var(--color-text-light);
  white-space: nowrap;
}

.search-field {
  flex: 1;
  display: flex;
  align-items: center;
  padding: 8px 24px;
  cursor: pointer;
  border-radius: 999px;
  transition: background 0.2s;
}

.search-field:hover {
  background: var(--color-background);
}

.field-icon {
  font-size: 1.5rem;
  margin-right: 16px;
  color: var(--color-primary);
}

.field-content {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  flex: 1;
}

.field-content label {
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  color: var(--color-text);
  margin-bottom: 2px;
  cursor: pointer;
  white-space: nowrap;
}

.search-input {
  border: none;
  background: transparent;
  width: 100%;
  font-size: 0.95rem;
  color: var(--color-text-light);
  outline: none;
  cursor: pointer;
  font-family: inherit;
}

.date-range {
  display: flex;
  align-items: center;
  gap: 4px;
  width: 100%;
}

.date-input {
  padding: 0;
  color: var(--color-text-light);
}

.date-separator {
  color: #ccc;
}

.select-input {
  appearance: none;
  -webkit-appearance: none;
  padding: 0;
}

.search-input:focus {
  color: var(--color-text);
}

.search-input::placeholder {
  color: #a0a0a0;
}

.field-divider {
  width: 1px;
  height: 32px;
  background-color: #ddd;
  margin: 0 8px;
}

.btn-search {
  background: var(--color-primary);
  color: white;
  border: none;
  border-radius: 999px;
  padding: 14px 32px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s;
}

.btn-search:hover {
  background: var(--color-primary-dark);
}

.btn-search:active {
  transform: scale(0.98);
}

.view-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 var(--spacing-lg) var(--spacing-xl);
}

.results-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--spacing-lg);
}

.results-header h2 {
  font-size: var(--font-size-xl);
  color: var(--color-text);
}

.results-count {
  color: var(--color-text-light);
  font-weight: 500;
}

.hotels-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: var(--spacing-xl);
}

.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: var(--spacing-md);
}

.empty-state h3 {
  font-size: var(--font-size-xl);
  color: var(--color-text);
  margin-bottom: var(--spacing-sm);
}

.empty-state p {
  color: var(--color-text-light);
  margin-bottom: var(--spacing-lg);
}
</style>
