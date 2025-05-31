<template>
  <div v-if="isLoading" class="ai-loading-overlay">
    <div class="ai-loading-inner">
      <img
        src="@/assets/ai-loading.gif"
        alt="AI 로딩 중"
        class="ai-loading-gif"
      />
      <div class="ai-loading-text">{{ animatedText }}</div>
      <button class="btn btn-outline-secondary mt-3" @click="goToThreadList">
        쓰레드 목록으로 이동
      </button>
    </div>
  </div>

  <div v-else class="thread-detail-wrapper">
    <div
      v-if="thread.cover"
      class="header-cover"
      :style="{ backgroundImage: `url(${thread.cover})` }"
      @click="showCoverModal = true"
    >
      <div class="header-gradient"></div>
    </div>

    <div class="thread-header-row">
      <h1 class="header-title">{{ thread.title }}</h1>
      <div v-if="thread.user" class="author-hover-area">
        <span class="author-meta">
          작성자:
          <strong class="author-nickname">{{ thread.user.nickname }}</strong>
        </span>
        <button class="popover-toggle-btn" @click="showPopover = !showPopover">
          <img
            src="@/assets/threads/home.png"
            alt="더보기"
            width="20"
            height="20"
          />
        </button>

        <transition name="slide-up">
          <div v-if="showPopover" class="popover-card">
            <RouterLink
              :to="{
                name: 'user-profile',
                params: { username: thread.user.username },
              }"
              class="btn btn-sm btn-outline-primary"
            >
              프로필 보기
            </RouterLink>
            <button
              v-if="accountStore.isLogin && !isMe"
              class="btn btn-sm"
              :class="isFollowed ? 'btn-outline-secondary' : 'btn-primary'"
              @click.stop="onFollow"
            >
              {{ isFollowed ? "언팔로우" : "팔로우" }}
            </button>
          </div>
        </transition>
      </div>
    </div>

    <div class="thread-detail-container">
      <div class="thread-main-row">
        <RouterLink
          v-if="thread.book"
          class="book-side-card"
          :to="{ name: 'book-detail', params: { bookId: thread.book.id } }"
        >
          <img :src="thread.book.cover" alt="책 표지" class="book-side-img" />
          <div>
            <div class="book-side-title">{{ thread.book.title }}</div>
            <div class="book-side-author">{{ thread.book.author }}</div>
          </div>
        </RouterLink>

        <div class="thread-content-area">
          <div class="thread-content-top">
            <div class="thread-content-text">
              <p class="thread-content">{{ thread.content }}</p>
              <div class="thread-meta">
                <strong>읽은 날짜:</strong>
                {{ formatDate(thread.read_date) || "미입력" }}
              </div>
            </div>
            <div class="thread-btns">
              <button
                class="like-button d-flex align-items-center gap-1 pb-1"
                @click="toggleLike"
              >
                <span>{{ thread.is_liked ? "❤️" : "🤍" }}</span>
                <span>좋아요 {{ thread.like_count }}</span>
              </button>
              <template v-if="canEdit">
                <RouterLink
                  :to="{ name: 'thread-edit', params: { threadId: thread.id } }"
                  class="btn btn-outline-primary me-2"
                >
                  수정
                </RouterLink>
                <button
                  class="btn btn-outline-danger"
                  @click="handleDeleteThread"
                >
                  삭제
                </button>
              </template>
            </div>
          </div>
        </div>
      </div>

      <teleport to="body">
        <transition name="fade">
          <div
            v-if="showCoverModal"
            class="modal-overlay"
            @click="showCoverModal = false"
          >
            <div class="modal-content" @click.stop>
              <img :src="thread.cover" alt="확대된 커버 이미지" />
            </div>
          </div>
        </transition>
      </teleport>

      <h2 class="mt-4">댓글 ({{ thread.comments?.length || 0 }})</h2>
      <CommentForm :threadId="thread.id" @comment-added="refreshComments" />
      <CommentList :comments="thread.comments" @refresh="refreshComments" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from "vue";
import { useRoute, useRouter } from "vue-router";
import { useThreadListStore } from "@/stores/threads";
import { useAccountStore } from "@/stores/accounts";
import CommentForm from "@/components/comment/CommentForm.vue";
import CommentList from "@/components/comment/CommentList.vue";
import Swal from "sweetalert2";

const router = useRouter();
const route = useRoute();
const threadStore = useThreadListStore();
const accountStore = useAccountStore();

const thread = ref(null);
const showPopover = ref(false);
const showCoverModal = ref(false);
const isLoading = ref(true);
const animatedText = ref("AI가 그림을 그리는 중입니다...");

const threadId = Number(route.params.threadId);
let pollTimer = null;
let textTimer = null;
let pollCount = 0;

const isMe = computed(
  () =>
    accountStore.myInfo &&
    thread.value?.user &&
    accountStore.myInfo.id === thread.value.user.id
);
const isFollowed = computed(() => thread.value?.user?.is_follow ?? false);
const canEdit = computed(
  () => thread.value?.user?.id === accountStore.myInfo?.id
);

const formatDate = (str) => str?.split("T")[0] ?? "";

const goToThreadList = () => router.push({ name: "threads" });

const onFollow = async () => {
  try {
    const res = await accountStore.followUser(thread.value.user.id);
    thread.value.user.is_follow = res.is_follow;
    if (res.is_follow) {
      Swal.fire({
        icon: "success",
        title: "팔로우 완료",
        timer: 1000,
        showConfirmButton: false,
      });
    } else {
      Swal.fire({
        icon: "info",
        title: "팔로우 취소 완료",
        timer: 1000,
        showConfirmButton: false,
      });
    }
  } catch (err) {
    Swal.fire({
      icon: "error",
      title: "팔로우 실패",
      text: "잠시 후 다시 시도해 주세요.",
    });
  }
};

const toggleLike = async () => {
  const res = await threadStore.toggleThreadLike(thread.value.id);
  thread.value.like_count = res.like_count;
  thread.value.is_liked = res.is_liked;
};

const handleDeleteThread = async () => {
  const confirm = await Swal.fire({
    title: "정말 삭제하시겠습니까?",
    text: "삭제하면 복구할 수 없습니다.",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "삭제",
    cancelButtonText: "취소",
    reverseButtons: true,
  });
  if (!confirm.isConfirmed) return;
  await threadStore.deleteThread(thread.value.id);
  await Swal.fire("삭제 완료", "", "success");
  router.push({ name: "threads" });
};

const refreshComments = async () => {
  thread.value = await threadStore.fetchThreadDetail(thread.value.id);
};

const handleEscClose = (e) => {
  if (e.key === "Escape") showCoverModal.value = false;
};

onMounted(async () => {
  window.addEventListener("keydown", handleEscClose);

  textTimer = setInterval(() => {
    const dotCount = ((animatedText.value.match(/\./g)?.length ?? 0) % 3) + 1;
    animatedText.value = "AI가 그림을 그리는 중입니다" + ".".repeat(dotCount);
  }, 500);

  try {
    const data = await threadStore.fetchThreadDetail(threadId);
    thread.value = data;

    if (!data.cover) {
      pollTimer = setInterval(async () => {
        const updated = await threadStore.fetchThreadDetail(threadId);
        thread.value = updated;
        pollCount++;
        if (updated.cover || pollCount >= 20) {
          clearInterval(pollTimer);
          isLoading.value = false;
        }
      }, 3000);
    } else {
      isLoading.value = false;
    }
  } catch (err) {
    console.error("로드 실패", err);
    await Swal.fire({
      title: "요청 실패",
      text: "쓰레드 로드 중 오류",
      icon: "error",
    });
    router.push({ name: "threads" });
  }
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", handleEscClose);
  if (pollTimer) clearInterval(pollTimer);
  if (textTimer) clearInterval(textTimer);
});
</script>

<style scoped>
.thread-detail-wrapper {
  position: relative;
  z-index: 1;
}

.author-hover-area {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

.thread-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  max-width: 1000px;
  margin: -55px auto 0;
  position: relative;
  z-index: 3;
  background: #fff;
  border-radius: 20px 20px 0 0;
  box-shadow: 0 4px 18px #0002;
  padding: 22px 34px 12px;
}

.header-title {
  font-size: 2.1rem;
  font-weight: 800;
  color: #232323;
  margin: 0;
}

.popover-card {
  position: absolute;
  bottom: 100%;
  right: 0;
  margin-bottom: 8px;
  background-color: white;
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 8px 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  z-index: 100;

  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  gap: 10px;

  min-width: 200px;
}
.popover-card .btn {
  min-width: 80px; /* "언팔로우" 기준 너비 고정 */
  text-align: center; /* "팔로우" 텍스트 가운데 정렬 */
  padding: 6px 10px;
}

/* 애니메이션 */
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all 0.2s ease;
}
.slide-up-enter-from,
.slide-up-leave-to {
  transform: translateY(6px);
  opacity: 0;
}
.slide-up-enter-to,
.slide-up-leave-from {
  transform: translateY(0);
  opacity: 1;
}

/* 커버 */
.header-cover {
  width: 100vw;
  height: 30vh;
  max-height: 330px;
  min-height: 260px;
  background-size: cover;
  background-position: center;
  position: relative;
  z-index: 1;
  overflow: hidden;
  cursor: pointer;
}

.header-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.7) 20%,
    rgba(0, 0, 0, 0.25) 70%,
    transparent 100%
  );
  z-index: 1;
}

.thread-detail-container {
  max-width: 1000px;
  margin: 0 auto 2.5rem;
  background: #fffaf5;
  border-radius: 0 0 18px 18px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.11);
  padding: 2rem 1.8rem 2.6rem;
  position: relative;
  z-index: 2;
  min-height: 420px;
}

.thread-main-row {
  display: flex;
  gap: 2rem;
  align-items: flex-start;
  flex-wrap: wrap;
}

.book-side-card {
  box-sizing: border-box;
  background: white;
  border-radius: 14px;
  padding: 1.2rem;
  min-width: 180px;
  max-width: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 0 2px 10px #0002;
  margin-bottom: 1rem;
  border: 1.5px solid #eee;
  cursor: pointer;
  text-decoration: none;
  transition: border-color 0.15s, box-shadow 0.15s, transform 0.13s;
}
.book-side-card:hover {
  outline: 2px solid #bbb;
  border-color: #eee; /* border는 그대로 두기 */
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
  transform: translateY(-2px);
}

.book-side-img {
  width: 90px;
  height: 120px;
  border-radius: 5px;
  object-fit: cover;
  margin-bottom: 0.8rem;
}

.book-side-title {
  font-weight: 600;
  font-size: 1.07rem;
  color: #464545;
  text-align: center;
  margin-bottom: 0.2rem;
}

.book-side-author {
  font-size: 0.93rem;
  color: #e0c38b;
  margin-bottom: 0.2rem;
  text-align: center;
}

.thread-content-area {
  flex: 1;
  min-width: 250px;
  max-width: 750px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 2px 10px #0002;
  border-radius: 10px;
}

.thread-content-top {
  display: flex;
  flex-direction: column;
  gap: 1.2rem;
  background: white;
  border-radius: 12px;
  padding: 1.5rem 2rem;
}

.thread-content-text {
  flex: 1;
}

.thread-content {
  font-size: 1.15rem;
  color: black;
  margin-bottom: 0.8rem;
  line-height: 1.7;
  word-break: break-word;
}

.thread-meta {
  font-size: 0.8rem;
  color: #858484;
}

.thread-btns {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  margin-top: 0.5rem;
}

.like-button {
  background: none;
  border: none;
  color: #ff6a85;
  font-size: 1.05rem;
  font-weight: bold;
  text-align: left;
  width: 120px;
}

.mt-4 {
  margin-top: 2.5rem;
}

.popover-toggle-btn {
  margin-left: 6px;
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 1.1rem;
  color: #555;
  padding: 2px 6px;
  transform: translateY(-5px);
  border: 0.5px solid;
}

.popover-toggle-btn:hover {
  color: #000;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.fade-enter-to,
.fade-leave-from {
  opacity: 1;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}
.modal-content {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  max-width: 70vw;
  max-height: 70vh;
}

.modal-content img {
  max-width: 80%;
  max-height: 80%;
  border-radius: 8px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
  cursor: default;
}
.close-btn {
  position: absolute;
  top: 8px;
  right: 12px;
  font-size: 2rem;
  color: white;
  background: transparent;
  border: none;
  cursor: pointer;
  z-index: 10;
}
.close-btn:hover {
  color: #ffaaaa;
}
.ai-loading-overlay {
  position: fixed;
  z-index: 9999;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.85);
  display: flex;
  align-items: center;
  justify-content: center;
}
.ai-loading-inner {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.ai-loading-gif {
  width: 160px;
  margin-bottom: 1rem;
}
.ai-loading-text {
  font-size: 1.3em;
  font-weight: bold;
  color: #003366;
  letter-spacing: 1px;
}
.ai-loading-inner button {
  padding: 0.5rem 1.25rem;
  font-size: 1rem;
}
</style>
