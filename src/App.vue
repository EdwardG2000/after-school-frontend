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