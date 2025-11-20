<template>
  <div>
    <!-- 🔵 NAVBAR -->
    <nav class="navbar">
      <div class="nav-left">
        <h1 class="logo">After-School Classes Booking</h1>
      </div>

      <div class="nav-right">
        <button class="cart-btn" @click="toggleCart">
          Cart 
          <span v-if="cart.length > 0" class="cart-count">{{ cart.length }}</span>
        </button>
      </div>
    </nav>

    <!-- PAGE CONTENT -->
    <div class="container">
      <Lessons 
        v-if="showLessons" 
        :lessons="sortedLessons" 
        @add-to-cart="addToCart"
      />

      <ShoppingCart 
        v-else 
        :cart="cart" 
        @remove-from-cart="removeFromCart"
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
        const valA = a[this.sortBy];
        const valB = b[this.sortBy];
        return this.sortOrder === "asc" ? (valA > valB ? 1 : -1) : (valA < valB ? 1 : -1);
      });
    }
  },
  methods: {
    async fetchLessons() {
      const res = await fetch("https://after-school-backend-3da9.onrender.com/lessons");
      this.lessons = await res.json();
    },
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        lesson.spaces--;
        this.cart.push(lesson);
      }
    },
    removeFromCart(lesson) {
      const index = this.cart.indexOf(lesson);
      if (index > -1) {
        this.cart.splice(index, 1);
        lesson.spaces++;
      }
    },
    toggleCart() {
      this.showLessons = !this.showLessons;
    },
    async checkout({ name, phone }) {
      if (!/^[a-zA-Z\s]+$/.test(name)) return alert("Name must contain letters.");
      if (!/^[0-9]+$/.test(phone)) return alert("Phone must contain numbers.");

      const order = { name, phone, lessons: this.cart };

      await fetch("https://after-school-backend-3da9.onrender.com/orders", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(order),
      });

      alert("Order submitted!");
      this.cart = [];
      this.showLessons = true;
    },
  },
  mounted() {
    this.fetchLessons();
  },
};
</script>

<style>
/* 🔷 NAVBAR */
.navbar {
  width: 100%;
  padding: 18px 28px;
  background: #1e3a8a; /* deep blue */
  color: white;
  display: flex;
  justify-content: space-between;
  align-items: center;

  box-shadow: 0 4px 16px rgba(0,0,0,0.15);
  position: sticky;
  top: 0;
  z-index: 10;
}

.logo {
  font-size: 1.35rem;
  font-weight: 700;
  margin: 0;
}

/* 🛒 CART BUTTON */
.cart-btn {
  background: #3b82f6;
  border: none;
  padding: 10px 16px;
  border-radius: 8px;
  color: white;
  font-weight: 600;
  cursor: pointer;
  position: relative;
}

.cart-btn:hover {
  background: #2563eb;
}

/* 🔴 SMALL CART BADGE */
.cart-count {
  background: red;
  color: white;
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 20px;
  margin-left: 8px;
}

/* Page container */
.container {
  max-width: 900px;
  margin: 30px auto;
  padding: 0 20px;
}
</style>