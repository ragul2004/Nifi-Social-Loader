<script setup>
import { ref } from 'vue'
import axios from 'axios'

const pageLinks = ref('')
const vkToken = ref('')
const instagramToken = ref('')
const pages = ref([])

const fetchPageInfo = async () => {
  try {
    const pageIds = pageLinks.value.split('\n').map(link => {
      const id = link.trim().replace(/^(https?:\/\/)?(www\.)?(vk\.com|instagram\.com)\//, '');
      const source = link.includes('vk.com') ? 'VK' : 'Instagram';
      return { id, source };
    });

    const response = await axios.post('http://localhost:8081/contentListener', {
      pages: pageIds,
      vkToken: vkToken.value,
      instagramToken: instagramToken.value
    }, {
      headers: {
        'Content-Type': 'application/json',
      }
    });

    pages.value = response.data.map(item => {
      if (item.error) {
        return {
          isError: true,
          platform: item.platform,
          query: item.query,
          errorMessage: item.error.error_msg
        };
      }
      return {
        isError: false,
        ...item.data,
        platform: item.platform
      };
    });
  } catch (error) {
    console.error('Error fetching page info:', error);
  }
};
</script>

<template>
  <div class="p-4">
    <div class="mb-4">
      <textarea
        v-model="pageLinks"
        placeholder="Enter VK or Instagram page links, each on a new line"
        class="border p-2 w-full mb-2"
        rows="4"
      ></textarea>
      <input
        v-model="vkToken"
        type="text"
        placeholder="Enter your VK token"
        class="border p-2 w-full mb-2"
      />
      <input
        v-model="instagramToken"
        type="text"
        placeholder="Enter your Instagram token"
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
        :key="page.id || page.query"
        class="border p-4 rounded shadow"
      >
        <template v-if="!page.isError">
          <img
            :src="page.photo_50"
            alt="Page icon"
            class="w-12 h-12 mb-2"
          />
          <h2 class="text-lg font-bold">{{ page.name }}</h2>
          <a
            :href="'https://' + page.platform + '.com/' + page.screen_name"
            target="_blank"
            class="text-blue-500"
          >
            Visit Page
          </a>
        </template>

        <template v-else>
          <h2 class="text-lg font-bold text-red-500">
            Error on platform {{ page.platform }} for page {{ page.query }}
          </h2>
          <p class="text-red-500">{{ page.errorMessage }}</p>
        </template>
      </div>
    </div>
  </div>
</template>

<style scoped>
</style>
