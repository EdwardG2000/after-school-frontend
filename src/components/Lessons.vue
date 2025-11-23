<template>
  <div class="lessons-grid">
    <article
      v-for="lesson in lessons"
      :key="lesson._id"
      class="lesson-card"
    >
      <div class="lesson-media">
        <img
          v-if="lesson.image"
          :src="lesson.image"
          :alt="`${lesson.subject} class image`"
        />
        <div v-else class="image-placeholder">
          <span>{{ lesson.subject }}</span>
        </div>
        <div class="lesson-tags">
          <span class="pill price-pill">£{{ lesson.price }}</span>
          <span
            class="pill spaces-pill"
            :class="{ full: lesson.spaces === 0 }"
          >
            {{ lesson.spaces === 0 ? 'Full' : `${lesson.spaces} spaces` }}
          </span>
        </div>
      </div>

      <div class="lesson-body">
        <div class="lesson-heading">
          <h3>{{ lesson.subject }}</h3>
          <p class="lesson-location">{{ lesson.location }}</p>
        </div>

        <button
          class="add-btn"
          @click="$emit('add-to-cart', lesson)"
          :disabled="lesson.spaces === 0"
        >
          {{ lesson.spaces === 0 ? 'Join waitlist' : 'Add to cart' }}
        </button>
      </div>
    </article>
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
.lessons-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
}

.lesson-card {
  background: linear-gradient(165deg, rgba(255, 255, 255, 0.08), rgba(18, 28, 72, 0.85));
  border-radius: 32px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.06);
  box-shadow: 0 25px 60px rgba(2, 7, 26, 0.45);
  display: flex;
  flex-direction: column;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.lesson-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 35px 80px rgba(5, 12, 30, 0.65);
}

.lesson-media {
  position: relative;
  min-height: 180px;
}

.lesson-media img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.image-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle at top, rgba(82, 230, 255, 0.4), rgba(18, 28, 72, 0.9));
  color: rgba(255, 255, 255, 0.8);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.lesson-tags {
  position: absolute;
  bottom: 16px;
  left: 16px;
  display: flex;
  gap: 8px;
}

.pill {
  padding: 6px 12px;
  border-radius: 999px;
  font-size: 0.8rem;
  font-weight: 600;
  backdrop-filter: blur(8px);
}

.price-pill {
  background: rgba(72, 111, 255, 0.95);
  color: #f5f7ff;
}

.spaces-pill {
  background: rgba(45, 212, 191, 0.92);
  color: #022c22;
}

.spaces-pill.full {
  background: rgba(244, 63, 94, 0.9);
  color: #fff5f7;
}

.lesson-body {
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.lesson-heading h3 {
  margin: 0;
  font-size: 1.35rem;
  color: #f8fbff;
}

.lesson-location {
  margin: 4px 0 0;
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
}

.lesson-meta {
  margin: 0;
  color: rgba(255, 255, 255, 0.65);
  font-size: 0.95rem;
  line-height: 1.4;
}

.add-btn {
  margin-top: auto;
  background: linear-gradient(120deg, #4c7dff, #52e6ff);
  color: #031432;
  border: none;
  border-radius: 18px;
  padding: 12px 18px;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.add-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 18px 30px rgba(82, 230, 255, 0.35);
}

.add-btn:disabled {
  background: rgba(255, 255, 255, 0.1);
  color: rgba(255, 255, 255, 0.6);
  cursor: not-allowed;
  box-shadow: none;
}
</style>