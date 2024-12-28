<script setup>
import { ref } from 'vue'
import axios from 'axios'

const pageLinks = ref('')
const token = ref('')
const pages = ref([])

const fetchPageInfo = async () => {
  try {
    const pageIds = pageLinks.value.split('\n').map(link => {
      // Trim and remove 'https://', 'http://', and 'vk.com/' if present
      return link.trim().replace(/^(https?:\/\/)?(www\.)?vk\.com\//, '')
    })
    const response = await axios.post('http://localhost:8081/contentListener', {
      source: 'VK',
      pageIds: pageIds
    }, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token.value}`
      }
    })
    pages.value = response.data.pages.map(page => {
      if (page.error) {
        return {
          ...page,
          errorMessage: `Error in page ${page.group_id}: ${page.error.error_msg}`
        }
      }
      return page
    })
  } catch (error) {
    console.error('Error fetching page info:', error)
  }
}
</script>

<template>
  <div class="p-4">
    <div class="mb-4">
      <textarea
        v-model="pageLinks"
        placeholder="Enter VK page links, each on a new line"
        class="border p-2 w-full mb-2"
        rows="4"
      ></textarea>
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
        <img v-if="!page.error" :src="page.photo_50" alt="Page icon" class="w-12 h-12 mb-2" />
        <h2 class="text-lg font-bold">{{ page.name }}</h2>
        <a
          v-if="!page.error"
          :href="'https://vk.com/' + page.screen_name"
          target="_blank"
          class="text-blue-500"
        >
          Visit Page
        </a>
        <p v-if="page.error" class="text-red-500">{{ page.errorMessage }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Tailwind CSS is used for styling, so no additional styles are needed here */
</style>
