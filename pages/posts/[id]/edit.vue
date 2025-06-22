<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'
import { useRouter, useRoute } from 'vue-router'
import { ref, reactive, onMounted } from 'vue'

const config = useRuntimeConfig()
const router = useRouter()
const route = useRoute()
const toast = useToast()

// ✅ Схема з правильним z.number()
const schema = z.object({
  title: z.string().min(1, 'Введіть заголовок'),
  excerpt: z.string().optional(),
  content_raw: z.string().optional(),
  category: z.object({ label: z.string(), value: z.number() }).nullable().optional(),
  is_published: z.boolean().optional()
})
type Schema = z.output<typeof schema>

// ✅ Формат дати
const formatDate = (dateString: string | null | undefined) => {
  if (!dateString) return '-'
  const date = new Date(dateString)
  return isNaN(date.getTime())
      ? '-'
      : new Intl.DateTimeFormat('uk-UA', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
      }).format(date)
}

// ✅ Початковий стан
const state = reactive<Partial<Schema> & {
  created_at?: string
  updated_at?: string
  published_at?: string
}>({
  title: '',
  excerpt: '',
  content_raw: '',
  category: null,
  is_published: false,
  created_at: '-',
  updated_at: '-',
  published_at: '-'
})

// ✅ Отримання категорій
const categoryOptions = ref<{ label: string; value: number }[]>([])

const fetchCategories = async () => {
  const res = await $fetch(`${config.public.apiBase}/blog/categories`)
  categoryOptions.value = res.data.map((cat: any) => ({
    label: cat.title,
    value: cat.id
  }))
}

// ✅ Отримання поста
const fetchPost = async () => {
  const post = await $fetch(`${config.public.apiBase}/blog/posts/${route.params.id}`)

  state.title = post.title ?? ''
  state.excerpt = post.excerpt ?? ''
  state.content_raw = post.content_raw ?? ''
  state.is_published = !!post.is_published // Приведення до boolean
  state.category = categoryOptions.value.find(cat => cat.value === post.category?.id) ?? null
  state.created_at = formatDate(post.created_at ?? null)
  state.updated_at = formatDate(post.updated_at ?? null)
  state.published_at = formatDate(post.published_at ?? null)
}

onMounted(async () => {
  await fetchCategories()
  await fetchPost()
})

const onSubmit = async (event: FormSubmitEvent<Schema>) => {
  await $fetch(`${config.public.apiBase}/blog/posts/${route.params.id}`, {
    method: 'PUT',
    body: {
      title: event.data.title,
      excerpt: event.data.excerpt,
      content_raw: event.data.content_raw,
      is_published: event.data.is_published,
      category_id: event.data.category?.value ?? null
    }
  })
  toast.add({ title: 'Пост оновлено', color: 'success' })
  router.push('/BlogPostsUi')
}
</script>


<template>
  <div class="max-w-xl mx-auto py-10 space-y-6">
    <h1 class="text-3xl font-bold">Редагувати пост</h1>

    <UForm :state="state" :schema="schema" @submit="onSubmit" class="space-y-4">
      <UFormField name="title" label="Заголовок">
        <UInput v-model="state.title" class="w-full" />
      </UFormField>

      <UFormField name="excerpt" label="Короткий опис">
        <UTextarea v-model="state.excerpt" rows="3" class="w-full" />
      </UFormField>

      <UFormField name="content_raw" label="Контент">
        <UTextarea v-model="state.content_raw" rows="6" class="w-full" />
      </UFormField>

      <UFormField name="category" label="Категорія">
        <USelectMenu
            v-model="state.category"
            :items="categoryOptions"
            placeholder="Оберіть категорію"
            clearable
            class="w-full"
        />
      </UFormField>

      <UFormField name="is_published" label="Опубліковано">
        <UCheckbox v-model="state.is_published" :model-value="state.is_published" />
      </UFormField>

      <UFormField name="published_at" label="Дата публікації">
        <UInput v-model="state.published_at" readonly class="w-full" />
      </UFormField>

      <UFormField name="created_at" label="Створено">
        <UInput v-model="state.created_at" readonly class="w-full" />
      </UFormField>

      <UFormField name="updated_at" label="Оновлено">
        <UInput v-model="state.updated_at" readonly class="w-full" />
      </UFormField>

      <UButton type="submit" label="Оновити" />
    </UForm>
  </div>
</template>
