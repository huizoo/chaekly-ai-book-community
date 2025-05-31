<template>
  <div class="container my-4">
    <h3 class="mb-4">📚 베스트셀러</h3>
    <div id="bestsellerCarousel" class="carousel slide" data-bs-ride="carousel" data-bs-interval="5000">
      <div class="carousel-inner">
        <div
          class="carousel-item"
          :class="{ active: index === 0 }"
          v-for="(chunk, index) in chunkedBooks"
          :key="index"
        >
          <div class="d-flex justify-content-between" style="height:380px;">
            <div
              class="card mx-2"
              v-for="book in chunk"
              :key="book.id"
              style="width: 18%; cursor: pointer"
              @click="goToDetail(book.id)"
            >
              <img :src="book.cover" class="card-img-top" alt="책 표지" />
              <div class="card-body text-center">
                <h6 class="card-title text-truncate mb-0 titletext">
                  {{ book.best_rank  }}위 · {{ book.title }}
                </h6>
              </div>
            </div>
          </div>
        </div>
      </div>

      <button class="carousel-control-prev" type="button" data-bs-target="#bestsellerCarousel" data-bs-slide="prev">
        <span class="carousel-control-prev-icon"></span>
      </button>
      <button class="carousel-control-next" type="button" data-bs-target="#bestsellerCarousel" data-bs-slide="next">
        <span class="carousel-control-next-icon"></span>
      </button>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios'
import { ref, onMounted, computed } from 'vue'
import { useRouter } from 'vue-router'

const bestBooks = ref([])
const router = useRouter()

onMounted(async () => {
  try {
    const res = await axios.get('http://127.0.0.1:8000/api/v1/books/bestsellers/')
    bestBooks.value = res.data
  } catch (err) {
    console.error('도서 목록 불러오기 실패', err)
  }
})

// 5개씩 나눠서 캐러셀 슬라이드 만들기
const chunkedBooks = computed(() => {
  const chunks = []
  for (let i = 0; i < bestBooks.value.length; i += 5) {
    chunks.push(bestBooks.value.slice(i, i + 5))
  }
  return chunks
})

const goToDetail = (bookId) => {
  router.push({ name: 'book-detail', params: { bookId } })
}
</script>

<style scoped>
.carousel-inner{
  height: 400px;
}

.card-title {
  font-size: 0.85rem;
}

.card-img-top {
  width: 100%;      /* 원하는 고정 너비 */
  height: 80%;
  object-fit: cover; /* 비율 맞춰서 잘림 */
  margin: 0 auto;    /* 카드 중앙 정렬 */
  border-radius: 6px;
  background: #f5f5f5;
}

.carousel-control-prev,
.carousel-control-next {
  filter: invert(100%);
}

.container {
  min-width: 1280px;
  margin: 0 auto;
}
#bestsellerCarousel {
  width: 100%;
}

</style>
