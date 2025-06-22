<script setup lang="ts">
import type { TableColumn } from '@nuxt/ui'
import { h } from 'vue'
import { useRouter } from 'vue-router'
import { useClipboard } from '@vueuse/core'

const router = useRouter()
const { copy } = useClipboard()

interface Category {
  id: number
  title: string
  slug: string
  description: string
  parent_id: number | null
}

interface CategoryResponse {
  data: Category[]
}

const config = useRuntimeConfig()
const categories = ref<Category[]>([])
const loading = ref(false)

const fetchCategories = async () => {
  loading.value = true
  try {
    const response = await $fetch<CategoryResponse>(`${config.public.apiBase}/blog/categories`)
    categories.value = response.data
  } finally {
    loading.value = false
  }
}

onMounted(fetchCategories)

function getDropdownActions(category: Category) {
  return [
    [
      {
        label: 'Редагувати',
        icon: 'i-lucide-edit',
        onSelect: () => router.push(`/categories/${category.id}/edit`)
      },
      {
        label: 'Видалити',
        icon: 'i-lucide-trash',
        color: 'error',
        onSelect: () => deleteCategory(category.id)
      }
    ]
  ]
}

const columns: TableColumn<Category>[] = [
  { accessorKey: 'id', header: '#' },
  { accessorKey: 'title', header: 'Назва' },
  { accessorKey: 'slug', header: 'Slug' },
  { accessorKey: 'description', header: 'Опис' },
  {
    accessorKey: 'actions',
    header: 'Дії',
    cell: () => null
  }
]

const deleteCategory = async (id: number) => {
  if (!confirm('Ви впевнені, що хочете видалити цю категорію?')) return

  try {
    await $fetch(`${config.public.apiBase}/blog/categories/${id}`, {
      method: 'DELETE'
    })
    categories.value = categories.value.filter(c => c.id !== id)
  } catch (e) {
    console.error('Помилка видалення категорії', e)
  }
}
</script>

<template>
  <div class="w-full max-w-screen-lg mx-auto space-y-4 pb-4">
    <div class="my-6 px-4 flex justify-between items-center">
      <h1 class="text-4xl font-bold leading-tight text-gray-900">Категорії</h1>
      <UButton type="submit" label="Додати категорію" icon="i-heroicons-plus" @click="router.push('/categories/create')" />
    </div>

    <UTable
        ref="table"
        :columns="columns"
        :data="categories || []"
        :loading="loading"
        class="table-auto w-full text-left"
    >
      <template #actions-cell="{ row }">
        <UDropdownMenu :items="getDropdownActions(row.original)">
          <UButton
              icon="i-lucide-ellipsis-vertical"
              color="neutral"
              variant="ghost"
              aria-label="Actions"
          />
        </UDropdownMenu>
      </template>
    </UTable>
  </div>
</template>
