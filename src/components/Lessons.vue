<template>
  <div class="lessons-wrapper">
    <div
      v-for="lesson in lessons"
      :key="lesson._id"
      class="lesson-card"
    >
      <img
        v-if="lesson.image"
        :src="lesson.image"
        :alt="`${lesson.subject} class image`"
        class="lesson-image"
      />

      <h3 class="lesson-title">{{ lesson.subject }}</h3>
      <p class="lesson-detail"><strong>Location:</strong> {{ lesson.location }}</p>
      <p class="lesson-detail"><strong>Price:</strong> £{{ lesson.price }}</p>
      <p class="lesson-detail"><strong>Spaces left:</strong> {{ lesson.spaces }}</p>

      <button
        class="add-btn"
        @click="$emit('add-to-cart', lesson)"
        :disabled="lesson.spaces === 0"
      >
        {{ lesson.spaces === 0 ? 'Full' : 'Add to Cart' }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "Lessons",
  props: {
    lessons: {
      type: Array,
      required: true,
    },
  },
};
</script>

<style scoped>
.lessons-wrapper {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}

/* Card styling */
.lesson-card {
  background: #ffffff;
  border-radius: 14px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0px 4px 16px rgba(0, 0, 0, 0.08);
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.lesson-card:hover {
  transform: translateY(-4px);
  box-shadow: 0px 8px 24px rgba(0, 0, 0, 0.12);
}

/* Image styling */
.lesson-image {
  width: 100%;
  height: 160px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 12px;
}

/* Title & text */
.lesson-title {
  font-size: 1.35rem;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 8px;
}

.lesson-detail {
  margin: 4px 0;
  font-size: 0.95rem;
  color: #444;
}

/* Button styling */
.add-btn {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 10px 18px;
  border-radius: 8px;
  cursor: pointer;
  margin-top: 12px;
  font-weight: 600;
  transition: background 0.15s ease;
}

.add-btn:hover {
  background: #2563eb;
}

.add-btn:disabled {
  background: #b3b3b3;
  cursor: not-allowed;
}
</style>