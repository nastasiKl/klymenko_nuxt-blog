<script setup lang="ts">
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'
import { useRouter, useRoute } from 'vue-router'
import { ref, reactive, onMounted } from 'vue'

const config = useRuntimeConfig()
const router = useRouter()
const route = useRoute()
const toast = useToast()

const schema = z.object({
  title: z.string().min(1, 'Введіть назву'),
  slug: z.string().optional(),
  description: z.string().optional(),
  parent: z.object({
    label: z.string(),
    value: z.number()
  }).nullable().optional(),
  created_at: z.string().optional(),
  updated_at: z.string().optional(),
})
type Schema = z.output<typeof schema>

const formatDate = (dateString: string | null | undefined) => {
  if (!dateString) return '-'
  const date = new Date(dateString)
  if (isNaN(date.getTime())) return '-'
  return new Intl.DateTimeFormat('uk-UA', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  }).format(date)
}

const state = reactive<Partial<Schema>>({
  title: '',
  slug: '',
  description: '',
  parent: null,
  created_at: '-',
  updated_at: '-',
})

const parentOptions = ref<{ label: string; value: number }[]>([])

const fetchParents = async () => {
  const res = await $fetch(`${config.public.apiBase}/blog/categories`)
  parentOptions.value = res.data.map((cat: any) => ({
    label: cat.title,
    value: cat.id
  }))
}

const fetchCategory = async () => {
  const data = await $fetch(`${config.public.apiBase}/blog/categories/${route.params.id}`)

  state.title = data.title
  state.slug = data.slug
  state.description = data.description ?? ''
  state.parent = parentOptions.value.find(cat => cat.value === data.parent_id) ?? null
  state.created_at = formatDate(data.created_at)
  state.updated_at = formatDate(data.updated_at)
}

onMounted(async () => {
  await fetchParents()
  await fetchCategory()
})

const onSubmit = async (event: FormSubmitEvent<Schema>) => {
  await $fetch(`${config.public.apiBase}/blog/categories/${route.params.id}`, {
    method: 'PUT',
    body: {
      title: event.data.title,
      slug: event.data.slug,
      description: event.data.description,
      parent_id: event.data.parent?.value ?? null
    }
  })
  toast.add({ title: 'Категорію оновлено', color: 'success' })
  router.push('/categories')
}
</script>

<template>
  <div class="max-w-xl mx-auto py-10 space-y-6">
    <h1 class="text-3xl font-bold">Редагувати категорію</h1>
    <UForm :state="state" :schema="schema" @submit="onSubmit" class="space-y-4">
      <UFormField name="title" label="Назва">
        <UInput v-model="state.title" class="w-full" />
      </UFormField>

      <UFormField name="slug" label="Псевдонім (необов'язково)">
        <UInput v-model="state.slug" class="w-full" />
      </UFormField>

      <UFormField name="description" label="Опис (необов'язково)">
        <UTextarea v-model="state.description" rows="4" class="w-full" />
      </UFormField>

      <UFormField name="parent" label="Батьківська категорія">
        <USelectMenu
            v-model="state.parent"
            :items="parentOptions"
            placeholder="Оберіть категорію"
            clearable
            class="w-full"
        />
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
