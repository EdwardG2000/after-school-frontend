<template>
  <div class="lessons-wrapper">
    <div 
      v-for="lesson in lessons" 
      :key="lesson._id" 
      class="lesson-card"
    >
      <h3 class="lesson-title">{{ lesson.subject }}</h3>

      <p class="lesson-detail">
        <strong>Location:</strong> {{ lesson.location }}
      </p>
      <p class="lesson-detail">
        <strong>Price:</strong> £{{ lesson.price }}
      </p>
      <p class="lesson-detail">
        <strong>Spaces left:</strong> {{ lesson.spaces }}
      </p>

      <button 
        class="add-btn"
        @click="$emit('add-to-cart', lesson)" 
        :disabled="lesson.spaces === 0"
      >
        {{ lesson.spaces === 0 ? "Sold Out" : "Add to Cart" }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ['lessons']
}
</script>

<style>
/* Outer wrapper to center layout */
.lessons-wrapper {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}

/* Modern card styling */
.lesson-card {
  background: #ffffff;
  border-radius: 14px;
  padding: 20px;
  margin-bottom: 20px;

  /* soft shadow */
  box-shadow: 0px 4px 16px rgba(0, 0, 0, 0.08);

  /* hover animation */
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.lesson-card:hover {
  transform: translateY(-4px);
  box-shadow: 0px 8px 24px rgba(0, 0, 0, 0.12);
}

/* Title Styling */
.lesson-title {
  font-size: 1.35rem;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 12px;
}

/* Text styling */
.lesson-detail {
  margin: 5px 0;
  font-size: 0.95rem;
  color: #444;
}

/* Button styling */
.add-btn {
  background: #3b82f6; /* blue */
  color: white;
  border: none;
  padding: 10px 18px;
  border-radius: 8px;
  cursor: pointer;
  margin-top: 12px;
  font-weight: 600;
  transition: background 0.15s ease;
}

/* Hover */
.add-btn:hover {
  background: #2563eb;
}

/* Disabled button */
.add-btn:disabled {
  background: #b3b3b3;
  cursor: not-allowed;
}
</style>