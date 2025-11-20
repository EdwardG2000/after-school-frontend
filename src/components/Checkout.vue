<template>
    <div>
      <h3>Checkout</h3>
      <form @submit.prevent="submitOrder">
        <input v-model="name" placeholder="Enter Name" required />
        <input v-model="phone" placeholder="Enter Phone" required />
        <button type="submit">Submit Order</button>
      </form>
    </div>
  </template>
  
  <script>
  export default {
    props: ['cart'],
    data() {
      return { name: '', phone: '' }
    },
    methods: {
      submitOrder() {
        this.$emit('checkout', {
          name: this.name,
          phone: this.phone
        })
      }
    }
  }
  </script>