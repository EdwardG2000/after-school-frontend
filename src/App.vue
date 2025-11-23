<template>
  <div class="app-shell">
    <header class="hero-card">
      <div class="hero-heading">
        <div>
          <p class="hero-eyebrow">After-School Classes Booking</p>
          <h1>After-School Classes Booking</h1>
        </div>

        <div class="hero-actions">
          <span class="hero-cart">Cart ({{ cart.length }} lessons)</span>
          <button class="primary-btn" @click="toggleBookingView">
            {{ showBooking ? 'Back to lessons' : 'Go to booking' }}
          </button>
        </div>
      </div>

    </header>

    <section v-if="!showBooking" class="panel surface">
      <div class="controls">
        <label class="field">
          <span>Sort by</span>
          <select v-model="sortBy">
            <option value="subject">Subject</option>
            <option value="location">Location</option>
            <option value="price">Price</option>
            <option value="spaces">Spaces</option>
          </select>
        </label>

        <label class="field">
          <span>Search</span>
          <input
            v-model.trim="searchTerm"
            type="text"
            placeholder="Try maths, science, coding…"
          />
        </label>

        <button class="order-btn" @click="toggleSortOrder">
          {{ sortOrderLabel }}
        </button>
      </div>

      <p v-if="fetchError" class="fetch-error">
        {{ fetchError }}
        <button type="button" class="link-btn" @click="fetchLessons">
          Try again
        </button>
      </p>

      <Lessons
        v-else
        :lessons="filteredLessons"
        @add-to-cart="addToCart"
      />

      <p v-if="!fetchError && filteredLessons.length === 0" class="empty-state">
        No lessons match your filters just yet — try clearing the search.
      </p>
    </section>

    <section v-else class="booking-view">
      <div class="booking-columns">
        <div class="panel">
          <div class="card-header">
            <div>
              <p class="eyebrow">Your selection</p>
              <h2>Shopping Cart</h2>
            </div>
            <span class="badge">{{ cart.length }}</span>
          </div>

          <ShoppingCart
            :cart="cart"
            @remove-from-cart="removeFromCart"
          />

          <div class="cart-summary" v-if="cart.length">
            <span>Total</span>
            <strong>£{{ cartTotal.toFixed(2) }}</strong>
          </div>
          <p class="empty-hint" v-else>
            Add a lesson to begin your booking.
          </p>
        </div>

        <BookingForm
          class="panel"
          :cart="cart"
          :loading="placingOrder"
          @book="checkout"
        />
      </div>
    </section>
  </div>
</template>

<script>
import Lessons from './components/Lessons.vue'
import ShoppingCart from './components/ShoppingCart.vue'
import BookingForm from './components/BookingForm.vue'

const LESSONS_ENDPOINT = 'https://after-school-backend-3da9.onrender.com/lessons'
const ORDERS_ENDPOINT = 'https://after-school-backend-3da9.onrender.com/orders'
const LESSON_IMAGES = {
  coding: 'public/coding image .png',
  english: 'public/english image.png',
  math: 'public/mathsimage.png',
  science: 'public/science image.png',
}

export default {
  components: { Lessons, ShoppingCart, BookingForm },
  data() {
    return {
      lessons: [],
      cart: [],
      showBooking: false,
      sortBy: 'subject',
      sortOrder: 'asc',
      searchTerm: '',
      placingOrder: false,
      fetchError: '',
    }
  },
  computed: {
    filteredLessons() {
      const query = this.searchTerm.trim().toLowerCase()
      const lessons = [...this.lessons].filter((lesson) => {
        if (!query) return true
        return `${lesson.subject} ${lesson.location}`
          .toLowerCase()
          .includes(query)
      })

      return lessons.sort((a, b) => {
        const valA = this.getSortableValue(a[this.sortBy])
        const valB = this.getSortableValue(b[this.sortBy])
        if (valA === valB) return 0
        if (this.sortOrder === 'asc') {
          return valA > valB ? 1 : -1
        }
        return valA < valB ? 1 : -1
      })
    },
    cartTotal() {
      return this.cart.reduce(
        (sum, lesson) => sum + Number(lesson.price || 0),
        0
      )
    },
    sortOrderLabel() {
      return this.sortOrder === 'asc' ? 'Ascending' : 'Descending'
    },
  },
  methods: {
    getSortableValue(value) {
      if (typeof value === 'string') {
        return value.toLowerCase()
      }
      const numberValue = Number(value)
      return Number.isNaN(numberValue) ? 0 : numberValue
    },
    async fetchLessons() {
      try {
        const response = await fetch(LESSONS_ENDPOINT)
        if (!response.ok) throw new Error('Failed to load lessons.')
        const lessons = await response.json()
        this.lessons = lessons.map((lesson) => {
          const subjectKey = (lesson.subject || '').trim().toLowerCase()
          const localImage = LESSON_IMAGES[subjectKey]
          return {
            ...lesson,
            image: localImage || lesson.image,
          }
        })
        this.fetchError = ''
      } catch (error) {
        console.error(error)
        this.fetchError =
          'We could not load the lessons right now. Please try again shortly.'
      }
    },
    addToCart(lesson) {
      if (!lesson || lesson.spaces <= 0) return
      lesson.spaces -= 1
      this.cart.push(lesson)
    },
    removeFromCart(lesson) {
      const index = this.cart.indexOf(lesson)
      if (index > -1) {
        this.cart.splice(index, 1)
        lesson.spaces += 1
      }
    },
    toggleBookingView() {
      this.showBooking = !this.showBooking
    },
    toggleSortOrder() {
      this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc'
    },
    async checkout(details) {
      if (!this.cart.length || this.placingOrder) return
      this.placingOrder = true
      try {
        const order = {
          ...details,
          lessons: this.cart.map((lesson) => ({
            id: lesson._id,
            subject: lesson.subject,
            price: lesson.price,
          })),
        }

        const response = await fetch(ORDERS_ENDPOINT, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(order),
        })

        if (!response.ok) {
          throw new Error('Failed to submit order.')
        }

        alert('Order submitted successfully!')
        this.cart = []
        this.showBooking = false
      } catch (error) {
        console.error(error)
        alert('We could not process your booking. Please try again.')
      } finally {
        this.placingOrder = false
      }
    },
  },
  mounted() {
    this.fetchLessons()
  },
}
</script>

<style>
* {
  box-sizing: border-box;
}

:root {
  --bg-start: #03050f;
  --bg-end: #010211;
  --panel: rgba(8, 16, 46, 0.92);
  --soft: rgba(255, 255, 255, 0.08);
  --accent: #3d7bff;
  --accent-2: #63d5ff;
  --text: #f6fbff;
  --muted: #a0b5d8;
}

body {
  margin: 0;
  font-family: 'Inter', 'Space Grotesk', system-ui, -apple-system, BlinkMacSystemFont,
    'Segoe UI', sans-serif;
  min-height: 100vh;
  background: radial-gradient(circle at top, rgba(36, 103, 255, 0.25), transparent 45%),
    linear-gradient(180deg, var(--bg-start), var(--bg-end) 65%);
  color: var(--text);
}

.app-shell {
  max-width: 1200px;
  margin: 0 auto;
  padding: 32px 24px 96px;
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.primary-btn {
  background: #0f172a;
  color: white;
  border: none;
  border-radius: 999px;
  padding: 12px 26px;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 12px 30px rgba(10, 16, 46, 0.35);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  white-space: nowrap;
}

.primary-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 20px 40px rgba(10, 16, 46, 0.45);
}

.hero-card {
  background: linear-gradient(135deg, #2a57ff, #31b9ff);
  border-radius: 40px;
  padding: 40px 48px;
  color: #f7fbff;
  box-shadow: 0 30px 120px rgba(8, 24, 68, 0.35);
}

.hero-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 24px;
}

.hero-eyebrow {
  margin: 0;
  text-transform: uppercase;
  letter-spacing: 0.35em;
  font-size: 0.68rem;
  opacity: 0.8;
}

.hero-card h1 {
  margin: 10px 0 4px;
  font-size: clamp(2rem, 4vw, 2.8rem);
}

.hero-copy {
  margin: 12px 0 0;
  max-width: 720px;
  font-size: 1rem;
  line-height: 1.7;
  color: rgba(255, 255, 255, 0.9);
}

.hero-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

.hero-cart {
  background: rgba(0, 0, 0, 0.25);
  padding: 10px 18px;
  border-radius: 999px;
  font-weight: 600;
}

.panel {
  background: var(--panel);
  border-radius: 34px;
  padding: 32px;
  border: 1px solid rgba(255, 255, 255, 0.04);
  box-shadow: 0 25px 60px rgba(2, 7, 26, 0.55);
}

.surface {
  display: flex;
  flex-direction: column;
  gap: 28px;
}

.controls {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  padding: 16px 18px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 18px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 0.85rem;
  letter-spacing: 0.02em;
  color: var(--muted);
}

.field select,
.field input {
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: rgba(3, 7, 25, 0.45);
  color: var(--text);
  padding: 12px 14px;
  font-size: 0.95rem;
  font-family: inherit;
}

.field input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.field select:focus,
.field input:focus {
  outline: none;
  border-color: rgba(255, 255, 255, 0.45);
  box-shadow: 0 0 0 3px rgba(99, 213, 255, 0.35);
}

.order-btn {
  align-self: flex-end;
  justify-self: flex-end;
  padding: 12px 20px;
  border-radius: 16px;
  border: none;
  font-weight: 600;
  cursor: pointer;
  background: rgba(255, 255, 255, 0.9);
  color: #16244b;
  box-shadow: inset 0 -2px 0 rgba(0, 0, 0, 0.2);
}

.fetch-error {
  background: rgba(251, 113, 133, 0.12);
  color: #fed7e2;
  border-radius: 16px;
  padding: 14px 18px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.link-btn {
  border: none;
  background: none;
  color: #8be4ff;
  font-weight: 600;
  cursor: pointer;
}

.empty-state {
  text-align: center;
  color: var(--muted);
  font-weight: 600;
}

.booking-view {
  width: 100%;
}

.booking-columns {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 24px;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin-bottom: 20px;
}

.card-header h2 {
  margin: 6px 0 0;
}

.eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.3em;
  font-size: 0.68rem;
  color: var(--muted);
  margin: 0;
}

.badge {
  background: rgba(255, 255, 255, 0.1);
  padding: 8px 16px;
  border-radius: 999px;
  font-weight: 600;
}

.cart-summary {
  margin-top: 24px;
  padding-top: 18px;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: var(--muted);
}

.cart-summary strong {
  font-size: 1.3rem;
  color: var(--text);
}

.empty-hint {
  color: var(--muted);
}

@media (max-width: 900px) {
  .glass-nav {
    flex-direction: column;
    align-items: flex-start;
  }
}

@media (max-width: 640px) {
  .hero-card {
    padding: 32px;
  }

  .hero-heading {
    flex-direction: column;
    align-items: flex-start;
  }

  .controls {
    grid-template-columns: 1fr;
  }
}
</style>