<template>
  <div>
    <ul>
      <li
        v-for="thread in visibleThreads"
        :key="thread.id"
        style="list-style: none; padding: 0; margin-bottom: 12px"
      >
        <ThreadItem :thread="thread" />
      </li>
    </ul>
  </div>
</template>

<script setup>
import { computed } from "vue";
import ThreadItem from '@/components/thread/ThreadItem.vue'
import { useAccountStore } from "@/stores/accounts.js";
const accountStore = useAccountStore();

const isMe = computed(() => {
  // 두 값이 모두 존재하고, id가 일치하면 내 프로필
  return (
    accountStore.myInfo &&
    accountStore.userProfile &&
    accountStore.myInfo.id === accountStore.userProfile.id
  );
});

const visibleThreads = computed(() => {
  if (!accountStore.userProfile) return [];
  if (isMe.value) {
    // 내 프로필: 전체 쓰레드
    return accountStore.userProfile.threads;
  }
  // 남의 프로필: 공개된 쓰레드만
  return accountStore.userProfile.threads.filter((thread) => thread.is_public);
});
</script>

<style scoped></style>
