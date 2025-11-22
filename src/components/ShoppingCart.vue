<template>
  <div class="cart-wrapper">
    <div v-if="cart.length === 0" class="empty-cart">
      <p>Your cart is empty.</p>
      <small>Add lessons to begin booking.</small>
    </div>

    <ul v-else class="cart-list">
      <li v-for="(lesson, index) in cart" :key="index" class="cart-line">
        <div>
          <p class="cart-lesson">{{ lesson.subject }}</p>
          <small>{{ lesson.location || 'After-school lesson' }}</small>
        </div>
        <div class="cart-price">
          <span>£{{ lesson.price }}</span>
          <button @click="$emit('remove-from-cart', lesson)">Remove</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ['cart'],
}
</script>

<style scoped>
.cart-wrapper {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.empty-cart {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 20px;
  padding: 20px;
  text-align: center;
  color: rgba(255, 255, 255, 0.75);
}

.cart-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.cart-line {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 18px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.06);
}

.cart-lesson {
  margin: 0;
  color: #f8fbff;
  font-weight: 600;
}

.cart-line small {
  color: rgba(255, 255, 255, 0.65);
}

.cart-price {
  display: flex;
  align-items: center;
  gap: 12px;
  font-weight: 600;
}

.cart-price button {
  border: none;
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.85);
  padding: 6px 12px;
  border-radius: 999px;
  cursor: pointer;
}
</style>