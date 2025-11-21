<template>
  <div>

    <!-- NAVBAR -->
    <nav class="navbar">
      <h1 class="app-title">After-School Classes Booking</h1>

      <button class="cart-btn" @click="toggleCart">
        Cart ({{ cart.length }})
      </button>
    </nav>

    <!-- MAIN CONTENT -->
    <div class="main-content">

      <!-- Show Lessons -->
      <Lessons 
        v-if="showLessons" 
        :lessons="sortedLessons" 
        @add-to-cart="addToCart"
      />

      <!-- Show Cart -->
      <ShoppingCart
        v-else
        :cart="cart"
        @remove="removeFromCart"
        @checkout="checkout"
      />
    </div>
  </div>
</template>

<script>
import Lessons from './components/Lessons.vue'
import ShoppingCart from './components/ShoppingCart.vue'
import Checkout from './components/Checkout.vue'

export default {
  components: { Lessons, ShoppingCart, Checkout },
  data() {
    return {
      lessons: [],
      cart: [],
      showLessons: true,
      sortBy: 'subject',
      sortOrder: 'asc',
    }
  },
  computed: {
    sortedLessons() {
      return [...this.lessons].sort((a, b) => {
        const valA = a[this.sortBy]
        const valB = b[this.sortBy]
        if (this.sortOrder === 'asc') return valA > valB ? 1 : -1
        else return valA < valB ? 1 : -1
      })
    },
  },
  methods: {
    async fetchLessons() {
      const res = await fetch('https://after-school-backend-3da9.onrender.com/lessons')
      this.lessons = await res.json()
    },
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        lesson.spaces--
        this.cart.push(lesson)
      }
    },
    removeFromCart(lesson) {
      const index = this.cart.indexOf(lesson)
      if (index > -1) {
        this.cart.splice(index, 1)
        lesson.spaces++
      }
    },
    toggleCart() {
      this.showLessons = !this.showLessons
    },
    async checkout({ name, phone }) {
      if (!/^[a-zA-Z\s]+$/.test(name)) {
        alert('Name must contain letters only.')
        return
      }
      if (!/^[0-9]+$/.test(phone)) {
        alert('Phone must contain numbers only.')
        return
      }

      const order = { name, phone, lessons: this.cart }

      await fetch('https://after-school-backend-3da9.onrender.com/orders', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(order),
      })

      alert('Order submitted successfully!')
      this.cart = []
      this.showLessons = true
    },
  },
  mounted() {
    this.fetchLessons()
  },
}
</script>

<style>
/* NAVBAR */
.navbar {
  background: #1e3a8a; /* Deep blue */
  color: white;
  padding: 18px 30px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.app-title {
  margin: 0;
  font-size: 1.4rem;
  font-weight: 700;
}

/* CART BUTTON */
.cart-btn {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 10px 16px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  transition: background 0.15s ease;
}

.cart-btn:hover {
  background: #2563eb;
}

/* MAIN CONTENT */
.main-content {
  max-width: 900px;
  margin: 25px auto;
}
</style>