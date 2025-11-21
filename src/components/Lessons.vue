<template>
 <template>
  <div>
    <div 
      v-for="lesson in lessons" 
      :key="lesson._id" 
      class="lesson-card"
    >
      <img :src="lesson.image" class="lesson-image" alt="Lesson image" />

      <h3>{{ lesson.subject }}</h3>
      <p>Location: {{ lesson.location }}</p>
      <p>Price: £{{ lesson.price }}</p>
      <p>Spaces left: {{ lesson.spaces }}</p>

      <button 
        @click="$emit('add-to-cart', lesson)" 
        :disabled="lesson.spaces === 0"
      >
        Add to Cart
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["lessons"],
};
</script>

<style>
.lesson-card {
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 15px;
  background: #fff;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.lesson-image {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 10px;
}
</style>
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
.lesson-card {
  border: 1px solid #ddd;
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 15px;
  background: #fff;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.lesson-image {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 10px;
  margin-bottom: 10px;
}
</style>