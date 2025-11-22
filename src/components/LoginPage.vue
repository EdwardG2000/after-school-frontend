<template>
  <section class="login-page">
    <div class="login-card">
      <p class="eyebrow">Welcome back</p>
      <h2>Access the booking hub</h2>
      <p>
        Sign in with your email to manage bookings, keep track of lessons, and
        securely invite family members.
      </p>

      <form class="login-form" @submit.prevent="handleSubmit">
        <label>
          Email
          <input v-model.trim="credentials.email" type="email" placeholder="you@email.com" required />
        </label>

        <label>
          Password
          <input v-model.trim="credentials.password" type="password" placeholder="••••••••" required minlength="6" />
        </label>

        <button type="submit">
          {{ busy ? 'Signing in…' : 'Continue to booking' }}
        </button>

        <p v-if="error" class="form-error">{{ error }}</p>
      </form>

      <p class="helper">
        Not ready to sign in? Continue as a demo account using <strong>demo@example.com</strong> / <strong>demo1234</strong>.
      </p>
    </div>
  </section>
</template>

<script>
export default {
  name: 'LoginPage',
  emits: ['login'],
  data() {
    return {
      credentials: {
        email: '',
        password: '',
      },
      error: '',
      busy: false,
    }
  },
  methods: {
    handleSubmit() {
      if (!this.credentials.email || !this.credentials.password) {
        this.error = 'Please provide both email and password.'
        return
      }

      this.error = ''
      this.busy = true
      setTimeout(() => {
        this.$emit('login', { email: this.credentials.email })
        this.busy = false
      }, 400)
    },
  },
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, rgba(37, 99, 235, 0.2), rgba(14, 165, 233, 0.08));
  padding: 40px 20px;
}

.login-card {
  background: white;
  border-radius: 28px;
  padding: 40px 48px;
  max-width: 420px;
  width: 100%;
  box-shadow: 0 30px 60px rgba(15, 23, 42, 0.25);
  border: 1px solid rgba(15, 23, 42, 0.08);
}

.eyebrow {
  letter-spacing: 0.5em;
  text-transform: uppercase;
  font-size: 0.7rem;
  color: #2563eb;
  margin: 0 0 8px;
}

h2 {
  margin: 0;
  font-size: 2rem;
}

.login-card p {
  color: #475569;
  line-height: 1.5;
  margin-top: 12px;
}

.login-form {
  margin-top: 24px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

label {
  display: flex;
  flex-direction: column;
  font-weight: 600;
  color: #0f172a;
  gap: 6px;
  font-size: 0.9rem;
}

input {
  border-radius: 12px;
  border: 1px solid rgba(15, 23, 42, 0.1);
  padding: 12px 14px;
  font-size: 0.95rem;
  font-family: inherit;
  transition: border 0.2s ease, box-shadow 0.2s ease;
}

input:focus {
  outline: none;
  border-color: #2563eb;
  box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.2);
}

button {
  background: linear-gradient(135deg, #2563eb, #1d4ed8);
  color: white;
  border: none;
  border-radius: 999px;
  padding: 12px 16px;
  font-weight: 600;
  font-size: 1rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

button:disabled {
  background: #94a3b8;
}

button:hover {
  transform: translateY(-1px);
  box-shadow: 0 14px 30px rgba(37, 99, 235, 0.35);
}

.form-error {
  color: #dc2626;
  font-weight: 600;
  margin: 0;
}

.helper {
  margin-top: 12px;
  font-size: 0.85rem;
  color: #475569;
}

@media (max-width: 520px) {
  .login-card {
    padding: 32px 24px;
  }
}
</style>

