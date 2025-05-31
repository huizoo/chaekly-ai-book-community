<template>
  <div class="calendar-wrap">
    <h4 class="mb-3">📅 읽은 도서 달력</h4>
    <v-calendar
      :attributes="calendarData"
      class="responsive-calendar"
      style="width: 100%; min-width: 0; max-width: 100%;"
    >
      <template #day-content="{ day, attributes }">
        <div>
          <span class="dayitem">{{ day.day }}</span>
          <div v-if="attributes && attributes.length">
            <div v-for="attr in attributes" :key="attr.key">
               <div v-html="attr.content.base.color" class="item"></div>
            </div>
          </div>
        </div>
      </template>
    </v-calendar>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { useAccountStore } from '@/stores/accounts'
import axios from 'axios'
import 'v-calendar/style.css'

const route = useRoute()
const accountStore = useAccountStore()
const username = route.params.username
const calendarData = ref([])

function truncateText(text, maxLength = 18) {
  if (text.length > maxLength) {
    return text.slice(0, maxLength) + '...'
  }
  return text
}

onMounted(async () => {
  try {
    const res = await axios.get(
      `http://127.0.0.1:8000/api/v1/users/${username}/calendar/`,
      {
        headers: {
          Authorization: `Token ${accountStore.token}`,
        }
      }
    )

    calendarData.value = res.data.map((item, i) => {
      let titleArr = Array.isArray(item.titles) ? item.titles : Object.values(item.titles)
      titleArr = titleArr.map(title =>
        typeof title === 'string'
          ? title
          : (title && typeof title === 'object' && 'title' in title
                ? title.title
                : JSON.stringify(title))
      );
      titleArr = titleArr.map(title => truncateText(title, 22))
      // 7. content 미리 확인
      let contentStr = titleArr.length > 1
        ? titleArr.map(title => `📖 ${title}`).join('<br>')
        : `📖 ${titleArr[0]}`

      return {
        key: item.date,
        dates: new Date(item.date),
        popover: {
          label: titleArr.length > 1
            ? titleArr.map(title => `- ${title}`).join('<br>')
            : `- ${titleArr[0]}`,
          visibility: 'hover'
        },
        content: contentStr
      }
    });
  } catch (err) {
    console.error(err)
  }
})
</script>

<style>
/* .calendar-wrap {
  width: 800px;
  margin: 0 auto;
  padding-right: 2rem;
} */

/* 달력 자체를 가로 100%로 */
.responsive-calendar {
  width: 100%;
}

.vc-weeks .vc-week {
  height: 100px;
}



.item {
  font-size: small;
}
</style>
