<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useHotelStore } from '../store/hotelStore'
import RoomCard from '../components/RoomCard.vue'
import BookingForm from '../components/BookingForm.vue'
import RoomDetailsModal from '../components/RoomDetailsModal.vue'

const route = useRoute()
const router = useRouter()
const { state, getRoomsForHotel, addReservation } = useHotelStore()

const hotelId = route.params.id as string
const hotel = computed(() => state.hotels.find(h => h.id === hotelId))
const rooms = computed(() => getRoomsForHotel(hotelId))

const selectedRoomId = ref<string | null>(null)
const selectedDetailsRoomId = ref<string | null>(null)
const bookingSuccess = ref(false)

const selectedRoom = computed(() => {
  return rooms.value.find(r => r.id === selectedRoomId.value)
})

const selectedDetailsRoom = computed(() => {
  return rooms.value.find(r => r.id === selectedDetailsRoomId.value)
})

const goBack = () => {
  router.push('/')
}

const handleBook = (roomId: string) => {
  selectedRoomId.value = roomId
  bookingSuccess.value = false
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

const handleShowDetails = (roomId: string) => {
  selectedDetailsRoomId.value = roomId
}

const closeDetailsModal = () => {
  selectedDetailsRoomId.value = null
}

const handleBookFromModal = (roomId: string) => {
  closeDetailsModal()
  handleBook(roomId)
}

const cancelBooking = () => {
  selectedRoomId.value = null
}

const submitBooking = (data: any) => {
  if (selectedRoomId.value) {
    addReservation({
      roomId: selectedRoomId.value,
      ...data,
      status: 'Pending'
    })
    bookingSuccess.value = true
    selectedRoomId.value = null
  }
}

onMounted(() => {
  if (!hotel.value) {
    router.push('/')
  }
})
</script>

<template>
  <div class="view-container" v-if="hotel">
    <div class="back-nav">
      <button class="btn-back" @click="goBack">
        ← Volver a resultados
      </button>
    </div>

    <header class="hotel-header">
      <img :src="hotel.image" :alt="hotel.name" class="hotel-hero-image" />
      <div class="hotel-header-content">
        <div class="badge city-badge">{{ hotel.city }}</div>
        <h1>{{ hotel.name }}</h1>
        <div class="rating">
          <span class="star">★</span>
          <span>{{ hotel.rating }} / 5</span>
        </div>
        <p class="hotel-desc">{{ hotel.description }}</p>
      </div>
    </header>
    
    <main class="view-content">
      
      <!-- Success Message -->
      <div v-if="bookingSuccess" class="alert alert-success animate-fade-in">
        <h3>¡Reserva Solicitada!</h3>
        <p>Tu solicitud de reserva para el <strong>{{ hotel.name }}</strong> ha sido enviada con éxito.</p>
        <button class="btn btn-primary" @click="bookingSuccess = false">Ver más habitaciones</button>
      </div>

      <!-- Booking Flow -->
      <div v-if="selectedRoom" class="booking-section animate-fade-in">
        <button class="btn-back" @click="cancelBooking">
          ← Volver a habitaciones
        </button>
        <BookingForm 
          :room="selectedRoom" 
          @cancel="cancelBooking"
          @submit="submitBooking"
        />
      </div>

      <!-- Rooms Listing -->
      <div v-else>
        <h2 class="section-title">Habitaciones Disponibles</h2>
        <div v-if="rooms.length > 0" class="rooms-grid">
          <RoomCard 
            v-for="room in rooms" 
            :key="room.id" 
            :room="room" 
            @book="handleBook"
            @show-details="handleShowDetails"
          />
        </div>
        <div v-else class="empty-state">
          <p>No hay habitaciones configuradas para este hotel actualmente.</p>
        </div>
      </div>
      
      <!-- Room Details Modal -->
      <RoomDetailsModal 
        v-if="selectedDetailsRoom"
        :room="selectedDetailsRoom"
        @close="closeDetailsModal"
        @book="handleBookFromModal"
      />
    </main>
  </div>
</template>

<style scoped>
.view-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: var(--spacing-lg);
}

.back-nav {
  margin-bottom: var(--spacing-md);
}

.btn-back {
  background: none;
  border: none;
  color: var(--color-primary);
  font-weight: 500;
  font-size: var(--font-size-md);
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  font-family: inherit;
}

.btn-back:hover {
  text-decoration: underline;
}

.hotel-header {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--spacing-xl);
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--spacing-xl);
}

@media (max-width: 768px) {
  .hotel-header {
    grid-template-columns: 1fr;
  }
}

.hotel-hero-image {
  width: 100%;
  height: 100%;
  min-height: 300px;
  object-fit: cover;
}

.hotel-header-content {
  padding: var(--spacing-xl);
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.city-badge {
  align-self: flex-start;
  background-color: var(--color-primary-light);
  color: var(--color-primary);
  margin-bottom: var(--spacing-sm);
}

.hotel-header-content h1 {
  font-size: var(--font-size-xxl);
  color: var(--color-text);
  margin-bottom: var(--spacing-xs);
  line-height: 1.1;
}

.rating {
  display: flex;
  align-items: center;
  gap: var(--spacing-xs);
  color: var(--color-text-light);
  font-size: var(--font-size-md);
  margin-bottom: var(--spacing-md);
}

.star {
  color: var(--color-warning);
  font-size: 1.2rem;
}

.hotel-desc {
  color: var(--color-text-light);
  font-size: var(--font-size-md);
  line-height: 1.6;
}

.section-title {
  font-size: var(--font-size-xl);
  margin-bottom: var(--spacing-lg);
  color: var(--color-text);
  border-bottom: 2px solid var(--color-background);
  padding-bottom: var(--spacing-xs);
}

.rooms-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: var(--spacing-xl);
}

.booking-section {
  max-width: 800px;
  margin: 0 auto;
}

.alert {
  padding: var(--spacing-lg);
  border-radius: var(--radius-md);
  margin-bottom: var(--spacing-xl);
  text-align: center;
}

.alert-success {
  background-color: var(--color-success-light);
  color: #065f46;
  border: 1px solid #34d399;
}

.alert-success h3 {
  margin-bottom: var(--spacing-sm);
}

.empty-state {
  text-align: center;
  padding: 3rem;
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  color: var(--color-text-light);
}
</style>
