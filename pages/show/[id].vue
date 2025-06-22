<script setup lang="ts">
import { useRoute, useRouter } from 'vue-router'

type Post = {
  id: number
  title: string
  slug: string
  is_published: boolean
  published_at: string | null
  user: { name: string }
  category: { title: string }
  content_html: string
}

const config = useRuntimeConfig()
const route = useRoute()
const router = useRouter()
const post = ref<Post | null>(null)
const loading = ref(true)

onMounted(async () => {
  try {
    const id = route.params.id
    post.value = await $fetch<Post>(`${config.public.apiBase}/blog/posts/${id}`)
  } finally {
    loading.value = false
  }
})

function goBack() {
  router.back()
}

</script>

<template>
  <div class="max-w-3xl mx-auto p-6 space-y-6">
    <UButton @click="goBack" color="gray" icon="i-heroicons-arrow-left" label="Назад" class="hover:bg-gray-100"/>

    <div v-if="loading">
      <USkeleton class="h-6 w-1/3 mb-2" />
      <USkeleton class="h-4 w-full" />
    </div>
    <div v-else-if="post">
      <h1 class="text-3xl font-bold leading-tight">{{ post.title }}</h1>

      <div class="text-sm text-gray-500">
        Автор <span class="font-medium">{{ post.user.name }}</span>
        | <span class="font-medium">"{{ post.category.title }}"</span>
        <br>
        {{ post.published_at || 'Чернетка' }}
      </div>

      <div
          class="prose prose-lg prose-gray max-w-none mt-6"
          v-html="post.content_html"
      />
    </div>

    <div v-else>
      <p class="text-red-500">Пост не знайдено.</p>
    </div>
  </div>
</template>
