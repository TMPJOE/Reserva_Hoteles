<script setup lang="ts">
import { ref, computed } from 'vue'
import type { Room } from '../store/hotelStore'

const props = defineProps<{
  room: Room
}>()

const emit = defineEmits<{
  (e: 'submit', data: { clientName: string; clientEmail: string; clientPhone: string; checkIn: string; checkOut: string; totalPrice: number }): void
  (e: 'cancel'): void
}>()

const clientName = ref('')
const clientEmail = ref('')
const clientPhone = ref('')
const checkIn = ref('')
const checkOut = ref('')

// Error Prevention (Nielsen): Validaciones
const minDate = computed(() => {
  const today = new Date()
  return today.toISOString().split('T')[0]
})

const checkoutMinDate = computed(() => {
  if (!checkIn.value) return minDate.value
  const ciDate = new Date(checkIn.value)
  ciDate.setDate(ciDate.getDate() + 1) // At least 1 night
  return ciDate.toISOString().split('T')[0]
})

const totalDays = computed(() => {
  if (!checkIn.value || !checkOut.value) return 0
  const start = new Date(checkIn.value)
  const end = new Date(checkOut.value)
  const diffTime = Math.abs(end.getTime() - start.getTime())
  return Math.ceil(diffTime / (1000 * 60 * 60 * 24))
})

const totalPrice = computed(() => {
  return totalDays.value * props.room.pricePerNight
})

const isValid = computed(() => {
  return clientName.value.trim() !== '' && 
         clientEmail.value.includes('@') && 
         clientPhone.value.trim().length >= 6 &&
         checkIn.value !== '' && 
         checkOut.value !== '' &&
         totalDays.value > 0
})

const handleSubmit = () => {
  if (isValid.value) {
    emit('submit', {
      clientName: clientName.value,
      clientEmail: clientEmail.value,
      clientPhone: clientPhone.value,
      checkIn: checkIn.value,
      checkOut: checkOut.value,
      totalPrice: totalPrice.value
    })
  }
}
</script>

<template>
  <div class="booking-form animate-fade-in">
    <div class="form-header">
      <h2>Completar Reserva</h2>
      <p>Estás reservando: <strong>{{ room.name }}</strong></p>
    </div>

    <form @submit.prevent="handleSubmit" class="form-body">
      <div class="input-group">
        <label class="input-label" for="name">Nombre Completo</label>
        <input 
          id="name"
          v-model="clientName" 
          type="text" 
          class="input-field" 
          placeholder="Ej: Ana García"
          required
        />
      </div>

      <div class="input-group">
        <label class="input-label" for="email">Correo Electrónico</label>
        <input 
          id="email"
          v-model="clientEmail" 
          type="email" 
          class="input-field" 
          placeholder="correo@ejemplo.com"
          required
        />
      </div>

      <div class="input-group">
        <label class="input-label" for="phone">Número de Teléfono</label>
        <input 
          id="phone"
          v-model="clientPhone" 
          type="tel" 
          class="input-field" 
          placeholder="Ej: +507 6000-0000"
          required
        />
      </div>

      <div class="date-row">
        <div class="input-group">
          <label class="input-label" for="checkin">Fecha de Entrada</label>
          <input 
            id="checkin"
            v-model="checkIn" 
            type="date" 
            class="input-field"
            :min="minDate"
            required
          />
        </div>
        <div class="input-group">
          <label class="input-label" for="checkout">Fecha de Salida</label>
          <input 
            id="checkout"
            v-model="checkOut" 
            type="date" 
            class="input-field"
            :min="checkoutMinDate"
            required
            :disabled="!checkIn"
          />
        </div>
      </div>

      <!-- Visibility of System Status (Nielsen): Real-time price calculation -->
      <div v-if="totalDays > 0" class="summary-box">
        <div class="summary-row">
          <span>Estancia ({{ totalDays }} noches x ${{ room.pricePerNight }})</span>
          <span>${{ totalPrice }}</span>
        </div>
        <div class="summary-total">
          <span>Total a Pagar</span>
          <span>${{ totalPrice }}</span>
        </div>
      </div>

      <div class="form-actions">
        <button type="button" class="btn btn-secondary" @click="emit('cancel')">Cancelar</button>
        <button type="submit" class="btn btn-primary" :disabled="!isValid">Confirmar y Pagar</button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.booking-form {
  background: var(--color-surface);
  padding: var(--spacing-xl);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  max-width: 500px;
  margin: 0 auto;
}

.form-header {
  margin-bottom: var(--spacing-lg);
  text-align: center;
}

.form-header h2 {
  color: var(--color-primary);
  margin-bottom: var(--spacing-xs);
}

.date-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: var(--spacing-md);
}

.summary-box {
  background: var(--color-background);
  padding: var(--spacing-md);
  border-radius: var(--radius-md);
  margin: var(--spacing-lg) 0;
  border: 1px dashed var(--color-primary-light);
}

.summary-row {
  display: flex;
  justify-content: space-between;
  color: var(--color-text-light);
  margin-bottom: var(--spacing-sm);
  font-size: var(--font-size-sm);
}

.summary-total {
  display: flex;
  justify-content: space-between;
  font-weight: 700;
  font-size: var(--font-size-lg);
  color: var(--color-primary);
  border-top: 1px solid #e2e8f0;
  padding-top: var(--spacing-sm);
}

.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: var(--spacing-md);
  margin-top: var(--spacing-lg);
}
</style>
