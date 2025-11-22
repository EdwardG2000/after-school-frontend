<template>
<form class="booking-form" @submit.prevent="handleSubmit">
    <div class="book-heading">
      <h3>Reserve a session</h3>
      <p class="book-subtitle">
        Confirm your spot before we fill up. We’ll only reach out if anything needs tweaking.
      </p>
    </div>

    <div class="details-grid">
      <label class="field">
        Name
        <input
          v-model.trim="form.name"
          type="text"
          placeholder="Parent / guardian name"
          required
        />
      </label>

      <label class="field">
        Phone
        <input
          v-model.trim="form.phone"
          type="tel"
          placeholder="E.g. 07912345678"
          required
        />
      </label>
    </div>

    <div class="details-grid">
      <label class="field">
        Preferred date
        <input v-model="form.date" type="date" :min="minDate" required />
      </label>

      <label class="field">
        Preferred time
        <select v-model="form.time" required>
          <option disabled value="">Pick a time slot</option>
          <option v-for="slot in timeSlots" :key="slot" :value="slot">
            {{ slot }}
          </option>
        </select>
      </label>
    </div>

    <section class="summary-card">
      <p class="booking-summary">
        Booking {{ cart.length }} lesson{{ cart.length === 1 ? '' : 's' }} · Total:
        £{{ total.toFixed(2) }}
      </p>
      <p class="summary-caption">
        We confirm availability for your preferred slot and follow up by phone.
      </p>
    </section>

    <button :disabled="loading">
      {{ loading ? 'Processing...' : 'Confirm booking' }}
    </button>
</form>
</template>

<script>
export default {
  name: 'BookingForm',
  props: {
    cart: {
      type: Array,
      required: true,
    },
    loading: {
      type: Boolean,
      default: false,
    },
  },
  emits: ['book'],
  data() {
    return {
      form: {
        name: '',
        phone: '',
        date: '',
        time: '',
      },
      timeSlots: ['15:30', '16:30', '17:30', '18:30'],
    }
  },
  computed: {
    minDate() {
      const tomorrow = new Date()
      tomorrow.setDate(tomorrow.getDate() + 1)
      return tomorrow.toISOString().split('T')[0]
    },
    total() {
      return this.cart.reduce((sum, lesson) => sum + Number(lesson.price), 0)
    },
  },
  methods: {
    handleSubmit() {
      this.$emit('book', { ...this.form })
    },
  },
}
</script>

<style scoped>
.booking-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
  background: rgba(255, 255, 255, 0.92);
  padding: 22px;
  border-radius: 22px;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.6),
    0 25px 50px rgba(15, 23, 42, 0.12);
  border: 1px solid rgba(15, 23, 42, 0.08);
}

.book-heading h3 {
  margin: 0;
  font-size: 1.35rem;
}

.book-subtitle {
  margin: 4px 0 0;
  color: #475569;
  font-size: 0.95rem;
}

label {
  font-weight: 600;
  color: #0f172a;
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 0.9rem;
}

input,
select {
  border-radius: 12px;
  border: 1px solid rgba(15, 23, 42, 0.15);
  padding: 12px 14px;
  font-size: 0.95rem;
  font-family: inherit;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  background: rgba(255, 255, 255, 0.8);
}

input:focus,
select:focus {
  outline: none;
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.2);
}

.details-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 14px;
}

.field input,
.field select {
  transition: border 0.2s ease, box-shadow 0.2s ease;
}

.summary-card {
  background: rgba(14, 165, 233, 0.12);
  border-radius: 16px;
  border: 1px solid rgba(14, 165, 233, 0.35);
  padding: 16px;
}

.summary-card .booking-summary {
  margin: 0;
  font-size: 1rem;
  color: #0f172a;
  font-weight: 600;
}

.summary-caption {
  margin: 6px 0 0;
  font-size: 0.9rem;
  color: #0f172a;
  opacity: 0.8;
}

button {
  margin-top: 4px;
  background: linear-gradient(135deg, #1d4ed8, #2563eb);
  color: white;
  border: none;
  border-radius: 999px;
  padding: 12px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 12px 24px rgba(37, 99, 235, 0.35);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

button:disabled {
  cursor: not-allowed;
  background: #94a3b8;
  box-shadow: none;
}

button:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 18px 32px rgba(37, 99, 235, 0.45);
}

@media (max-width: 600px) {
  .details-grid {
    grid-template-columns: 1fr;
  }
}
</style>

