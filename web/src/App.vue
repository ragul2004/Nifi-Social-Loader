<script setup>
import { ref } from 'vue'
import axios from 'axios'

const pageLinks = ref('')
const token = ref('')
const pages = ref([])

const fetchPageInfo = async () => {
  try {
    const response = await axios.post('http://localhost:8081/contentListener', {
      source: 'VK',
      pageIds: pageLinks.value.split(',').map(link => link.trim())
    }, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token.value}`
      }
    })
    pages.value = response.data.pages
  } catch (error) {
    console.error('Error fetching page info:', error)
  }
}
</script>

<template>
  <div class="p-4">
    <div class="mb-4">
      <input
        v-model="pageLinks"
        type="text"
        placeholder="Enter VK page links, separated by commas"
        class="border p-2 w-full mb-2"
      />
      <input
        v-model="token"
        type="text"
        placeholder="Enter your token"
        class="border p-2 w-full mb-2"
      />
      <button
        @click="fetchPageInfo"
        class="bg-blue-500 text-white p-2 rounded"
      >
        Fetch Page Info
      </button>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <div
        v-for="page in pages"
        :key="page.id"
        class="border p-4 rounded shadow"
      >
        <img :src="page.photo_50" alt="Page icon" class="w-12 h-12 mb-2" />
        <h2 class="text-lg font-bold">{{ page.name }}</h2>
        <a
          :href="'https://vk.com/' + page.screen_name"
          target="_blank"
          class="text-blue-500"
        >
          Visit Page
        </a>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Tailwind CSS is used for styling, so no additional styles are needed here */
</style>
