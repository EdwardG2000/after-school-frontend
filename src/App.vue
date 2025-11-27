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

</style>