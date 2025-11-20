<template>
    <div>
      <h2>Shopping Cart</h2>
      <div v-if="cart.length === 0">
        <p>Your cart is empty.</p>
      </div>
      <div v-else>
        <div v-for="(lesson, index) in cart" :key="index">
          <p>{{ lesson.subject }} - £{{ lesson.price }}</p>
          <button @click="$emit('remove-from-cart', lesson)">Remove</button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: ['cart']
  }
  </script>