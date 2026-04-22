<script setup lang="ts">
import { useHotelStore } from '../store/hotelStore'
import ReservationList from '../components/ReservationList.vue'

const { state, confirmReservation, cancelReservation } = useHotelStore()

const handleConfirm = (id: string) => {
  if (confirm(`¿Estás seguro de confirmar la reserva ${id}?`)) {
    confirmReservation(id)
  }
}

const handleCancel = (id: string) => {
  if (confirm(`ATENCIÓN: ¿Estás seguro de cancelar la reserva ${id}? Esta acción liberará la habitación.`)) {
    cancelReservation(id)
  }
}
</script>

<template>
  <div class="view-container">
    <header class="view-header">
      <h1>Panel de Recepción</h1>
      <p>Gestión diaria de reservas y ocupación</p>
    </header>
    
    <main class="view-content">
      <div class="reservations-section">
        <div class="section-header">
          <h2>Todas las Reservaciones</h2>
          <span class="badge badge-primary">{{ state.reservations.length }} Totales</span>
        </div>
        
        <ReservationList 
          :reservations="state.reservations"
          :show-actions="true"
          @confirm="handleConfirm"
          @cancel="handleCancel"
        />
      </div>
    </main>
  </div>
</template>

<style scoped>
.view-container {
  padding: var(--spacing-lg);
  max-width: 1200px;
  margin: 0 auto;
}

.view-header {
  margin-bottom: var(--spacing-xl);
}

.view-header h1 {
  font-size: var(--font-size-xxl);
  color: var(--color-primary);
  margin-bottom: var(--spacing-xs);
  letter-spacing: -0.02em;
}

.view-header p {
  color: var(--color-text-light);
  font-size: var(--font-size-md);
}

.reservations-section {
  background: var(--color-surface);
  padding: var(--spacing-lg);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--spacing-md);
}

.section-header h2 {
  font-size: var(--font-size-lg);
  color: var(--color-text);
}

.badge-primary {
  background-color: var(--color-primary-light);
  color: var(--color-primary);
  font-weight: 600;
}
</style>
