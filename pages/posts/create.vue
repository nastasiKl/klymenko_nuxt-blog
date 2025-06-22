<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'
import { useRouter } from 'vue-router'
import { ref, reactive, onMounted } from 'vue'

const config = useRuntimeConfig()
const router = useRouter()
const toast = useToast()

const schema = z.object({
  title: z.string().min(1, 'Введіть заголовок'),
  excerpt: z.string().optional(),
  content_raw: z.string().optional(),
  category: z.object({ label: z.string(), value: z.number() }).nullable().optional(),
  is_published: z.boolean().optional()
})
type Schema = z.output<typeof schema>

const state = reactive<Partial<Schema>>({
  title: '',
  excerpt: '',
  content_raw: '',
  category: null,
  is_published: false
})

const categoryOptions = ref<{ label: string; value: number }[]>([])

const fetchCategories = async () => {
  const res = await $fetch(`${config.public.apiBase}/blog/categories`)
  categoryOptions.value = res.data.map((cat: any) => ({
    label: cat.title,
    value: cat.id
  }))
}

onMounted(fetchCategories)

const onSubmit = async (event: FormSubmitEvent<Schema>) => {
  await $fetch(`${config.public.apiBase}/blog/posts`, {
    method: 'POST',
    body: {
      title: event.data.title,
      excerpt: event.data.excerpt,
      content_raw: event.data.content_raw,
      is_published: event.data.is_published,
      category_id: event.data.category?.value ?? null
    }
  })
  toast.add({ title: 'Пост створено', color: 'success' })
  router.push('/BlogPostsUi')
}
</script>

<template>
  <div class="max-w-xl mx-auto py-10 space-y-6">
    <h1 class="text-3xl font-bold">Створити новий пост</h1>

    <UForm :state="state" :schema="schema" @submit="onSubmit" class="space-y-4">
      <UFormField name="title" label="Заголовок">
        <UInput v-model="state.title" class="w-full" />
      </UFormField>

      <UFormField name="excerpt" label="Короткий опис">
        <UTextarea v-model="state.excerpt" :rows="3" class="w-full" />
      </UFormField>

      <UFormField name="content_raw" label="Контент">
        <UTextarea v-model="state.content_raw" :rows="6" class="w-full" />
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
        <UCheckbox v-model="state.is_published" />
      </UFormField>

      <UButton type="submit" label="Створити пост" />
    </UForm>
  </div>
</template>
