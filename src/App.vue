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

const BASE_URL = 'https://after-school-backend-3da9.onrender.com'
const LESSONS_ENDPOINT = `${BASE_URL}/lessons`
const ORDERS_ENDPOINT = `${BASE_URL}/orders`

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
        return `${lesson.subject} ${lesson.location} ${lesson.price} ${lesson.spaces}`
          .toLowerCase()
          .includes(query)
      })

      return lessons.sort((a, b) => {
        const valA = this.getSortableValue(a[this.sortBy])
        const valB = this.getSortableValue(b[this.sortBy])
        if (valA === valB) return 0
        return this.sortOrder === 'asc'
          ? valA > valB ? 1 : -1
          : valA < valB ? 1 : -1
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
      if (typeof value === 'string') return value.toLowerCase()
      const numberValue = Number(value)
      return Number.isNaN(numberValue) ? 0 : numberValue
    },

    async fetchLessons() {
      try {
        const response = await fetch(LESSONS_ENDPOINT)
        if (!response.ok) throw new Error('Failed to load lessons.')

        const lessons = await response.json()

        this.lessons = lessons.map((lesson) => ({
          ...lesson,
          image: `${BASE_URL}/lesson-images/${lesson.image}`,
        }))

        this.fetchError = ''

      } catch (error) {
        console.error(error)
        this.fetchError = 'We could not load the lessons right now.'
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
            spaces: lesson.spaces,
          })),
        }

        const response = await fetch(ORDERS_ENDPOINT, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(order),
        })

        if (!response.ok) throw new Error('Failed to submit order.')
        for (const lesson of this.cart) {
          await fetch(`${LESSONS_ENDPOINT}/${lesson._id}`, {
            method: 'PUT',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ spaces: lesson.spaces }),
          })
        }

        alert('Order submitted and spaces updated!')
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
:root {
  font-family: 'Inter', 'SF Pro Display', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color: #e2e8f0;
  background-color: #030715;
}


*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  min-height: 100vh;
  background: radial-gradient(circle at top, rgba(82, 230, 255, 0.25), rgba(3, 7, 21, 0.95));
  color: inherit;
}

:root {
  font-family: 'Inter', 'SF Pro Display', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color: #e2e8f0;
  background-color: #030715;
}

.app-shell {
  padding: clamp(20px, 4vw, 48px);
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.hero-card {
  background: linear-gradient(145deg, rgba(11, 21, 54, 0.95), rgba(33, 14, 68, 0.9));
  border-radius: 36px;
  padding: clamp(24px, 4vw, 48px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 40px 120px rgba(3, 7, 21, 0.65);
}

.hero-heading {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 24px;
  flex-wrap: wrap;
}

.hero-eyebrow {
  letter-spacing: 0.4em;
  text-transform: uppercase;
  font-size: 0.7rem;
  margin: 0;
  color: rgba(82, 230, 255, 0.85);
}

.hero-heading h1 {
  margin: 8px 0 0;
  font-size: clamp(1.8rem, 4vw, 3rem);
  color: #f8fbff;
}

.hero-actions {
  display: flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

.hero-cart {
  font-weight: 600;
  color: rgba(255, 255, 255, 0.75);
}

.primary-btn {
  border: none;
  border-radius: 999px;
  padding: 12px 20px;
  font-weight: 600;
  font-size: 0.95rem;
  cursor: pointer;
  background: linear-gradient(120deg, #4c7dff, #59f2ff);
  color: #051029;
  box-shadow: 0 18px 36px rgba(82, 230, 255, 0.35);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.primary-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 25px 60px rgba(82, 230, 255, 0.45);
}

.panel {
  border-radius: 30px;
  padding: clamp(20px, 3vw, 32px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  background: rgba(9, 14, 35, 0.85);
  box-shadow: 0 35px 80px rgba(5, 9, 25, 0.45);
}

.surface {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.controls {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 18px;
  align-items: flex-end;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.85);
}

.field select,
.field input {
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.15);
  background: rgba(255, 255, 255, 0.08);
  color: #f8fbff;
  padding: 10px 16px;
  font-size: 0.95rem;
  font-family: inherit;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.field input::placeholder {
  color: rgba(255, 255, 255, 0.4);
}

.field input:focus,
.field select:focus {
  outline: none;
  border-color: rgba(99, 213, 255, 0.85);
  box-shadow: 0 0 0 2px rgba(82, 230, 255, 0.3);
}

.order-btn {
  justify-self: start;
  border-radius: 999px;
  padding: 12px 18px;
  border: none;
  cursor: pointer;
  font-weight: 600;
  color: #051029;
  background: rgba(255, 255, 255, 0.92);
  transition: background 0.2s ease, transform 0.2s ease;
}

.order-btn:hover {
  background: #ffffff;
  transform: translateY(-1px);
}

.fetch-error {
  background: rgba(239, 68, 68, 0.15);
  border: 1px solid rgba(248, 113, 113, 0.4);
  color: #fecaca;
  padding: 14px 16px;
  border-radius: 18px;
  margin: 0;
}

.link-btn {
  background: none;
  border: none;
  color: #7dd3fc;
  font-weight: 600;
  cursor: pointer;
  text-decoration: underline;
  margin-left: 8px;
}

.empty-state {
  margin: 0;
  padding: 18px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.04);
  text-align: center;
  color: rgba(255, 255, 255, 0.75);
}

.booking-view {
  display: flex;
  flex-direction: column;
  gap: 24px;
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
  margin-bottom: 16px;
}

.card-header .eyebrow {
  text-transform: uppercase;
  letter-spacing: 0.35em;
  font-size: 0.7rem;
  margin: 0;
  color: rgba(82, 230, 255, 0.8);
}

.card-header h2 {
  margin: 6px 0 0;
  color: #f8fbff;
}

.badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 36px;
  height: 36px;
  border-radius: 999px;
  background: rgba(82, 230, 255, 0.2);
  color: #52e6ff;
  font-weight: 700;
}

.cart-summary {
  margin-top: 20px;
  padding: 16px 20px;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.06);
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-weight: 600;
}

.empty-hint {
  margin: 16px 0 0;
  color: rgba(255, 255, 255, 0.65);
  text-align: center;
}

@media (max-width: 640px) {
  .hero-heading {
    flex-direction: column;
    align-items: flex-start;
  }

  .controls {
    grid-template-columns: 1fr;
  }
}
</style>