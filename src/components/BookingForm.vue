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
  background: rgba(4, 8, 26, 0.85);
  padding: 24px;
  border-radius: 26px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 25px 50px rgba(2, 7, 26, 0.55);
  color: rgba(255, 255, 255, 0.9);
}

.book-heading h3 {
  margin: 0;
  font-size: 1.35rem;
  color: #f8fbff;
}

.book-subtitle {
  margin: 4px 0 0;
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.95rem;
}

label {
  font-weight: 600;
  color: rgba(255, 255, 255, 0.8);
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-size: 0.9rem;
}

input,
select {
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.18);
  padding: 12px 14px;
  font-size: 0.95rem;
  font-family: inherit;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
  background: rgba(255, 255, 255, 0.08);
  color: #f8fbff;
}

input::placeholder {
  color: rgba(255, 255, 255, 0.45);
}

input:focus,
select:focus {
  outline: none;
  border-color: rgba(99, 213, 255, 0.8);
  box-shadow: 0 0 0 2px rgba(82, 230, 255, 0.25);
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
  background: rgba(83, 130, 255, 0.12);
  border-radius: 18px;
  border: 1px solid rgba(255, 255, 255, 0.08);
  padding: 16px;
}

.summary-card .booking-summary {
  margin: 0;
  font-size: 1rem;
  color: #f8fbff;
  font-weight: 600;
}

.summary-caption {
  margin: 6px 0 0;
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);
  opacity: 0.8;
}

button {
  margin-top: 4px;
  background: linear-gradient(120deg, #4c7dff, #52e6ff);
  color: #06102c;
  border: none;
  border-radius: 999px;
  padding: 14px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 12px 24px rgba(82, 230, 255, 0.35);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

button:disabled {
  cursor: not-allowed;
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.6);
  box-shadow: none;
}

button:not(:disabled):hover {
  transform: translateY(-2px);
  box-shadow: 0 18px 32px rgba(82, 230, 255, 0.5);
}

@media (max-width: 600px) {
  .details-grid {
    grid-template-columns: 1fr;
  }
}
</style>

