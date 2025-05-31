<template>
  <RouterLink
    :to="{ name: 'book-detail', params: { bookId: book.id } }"
    class="d-flex p-3 book-card"
    style="text-decoration: none; color: inherit"
  >
    <img :src="book.cover" class="book-cover me-3" alt="cover" />
    <div class="flex-grow-1">
      <h5 class="card-title mb-2">
        {{ book.title }}
      </h5>
      <p class="card-text mb-1"><strong>저자:</strong> {{ book.author }}</p>
      <p class="card-text mb-1">
        <strong>출판사:</strong> {{ book.publisher }}
      </p>
      <p class="card-text desc-text mb-0">
        <strong>설명:</strong> {{ book.description }}
      </p>
    </div>
  </RouterLink>
</template>

<script setup>
import { RouterLink } from "vue-router";

const props = defineProps({
  book: Object,
});
</script>

<style scoped>
.book-cover {
  width: 100px;
  height: 150px;
  object-fit: cover;
}
.desc-text {
  max-height: 3.6em;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.book-card {
  background-color: #fff;
  border: 1.5px solid #e0e0e0;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.03);
  transition: border-color 0.2s, box-shadow 0.2s, transform 0.2s;
  overflow: hidden;
}

.book-card:hover {
  border-color: #888;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.14);
  transform: translateY(-5px);
}
</style>
