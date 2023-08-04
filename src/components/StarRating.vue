<!-- StarRating.vue (Star Rating Component) -->
<template>
    <div class="star-rating">
      <span v-for="i in Math.round(rating)" :key="i" class="star">&#9733;</span>
    </div>
  </template>
  
  <script>
  
  export default {
    props: {
      rating: {
        type: Number,
        required: true,
      },
    },
  };
  </script>
  
  <style scoped>
  .star-rating {
    font-size: 24px; /* Adjust the size of the stars */
    color: gold; /* Change the color of the stars */
  }
  </style>
  