<template>
  <div>
    <div v-if="wishlistBooks.length">
      <ul class="book-list">
        <li v-for="book in wishlistBooks" :key="book.id" class="book-item">
          <img :src="book.cover" alt="cover" class="cover" />
          <span>{{ book.title }}</span>
        </li>
      </ul>
    </div>
    <p v-else>서재에 담긴 책이 없습니다.</p>
  </div>
</template>

<script setup>
import { computed } from "vue";
import { useAccountStore } from "@/stores/accounts.js";
const accountStore = useAccountStore();

const MEDIA_URL = "http://127.0.0.1:8000"; // 환경에 따라 변경

const wishlistBooks = computed(() => {
  // userProfile.wishlist가 서버에서 전달되는 위시리스트 책 배열이라고 가정
  return accountStore.userProfile?.wishlist || [];
});

</script>

<style scoped>
.book-list {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  padding: 0;
  list-style: none;
}

.book-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 120px;
}

.cover {
  width: 100px;
  height: 150px;
  object-fit: cover;
  border-radius: 8px;
  margin-bottom: 8px;
  border: 1px solid #eee;
}
</style>
