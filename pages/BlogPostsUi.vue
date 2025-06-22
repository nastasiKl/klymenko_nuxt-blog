<script setup lang="ts">
import { getPaginationRowModel } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'
import { h, shallowRef, computed, ref, watch } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

type Post = {
  id: number
  title: string
  slug: string
  is_published: boolean
  published_at: string | null
  user_id: number
  category_id: number
  user: { name: string }
  category: { title: string }
}

type PostResponse = {
  data: Post[]
  meta: {
    total: number
    current_page: number
    last_page: number
    per_page: number
  }
}

const config = useRuntimeConfig()

const pagination = ref({
  pageIndex: 0,
  pageSize: 9
})

const currentPage = computed(() => pagination.value.pageIndex + 1)
const pageSize = computed(() => pagination.value.pageSize)
const total = ref(0)
const data = shallowRef<Post[]>([])
const loading = ref(false)

const fetchPosts = async () => {
  loading.value = true
  const response = await $fetch<PostResponse>(
      `${config.public.apiBase}/blog/posts?page=${currentPage.value}&per_page=${pageSize.value}`
  )
  total.value = response.meta.total
  data.value = response.data
  loading.value = false
}

watch([currentPage, pageSize], fetchPosts, { immediate: true })

const deletePost = async (id: number) => {
  if (!confirm('Ви впевнені, що хочете видалити цей пост?')) return
  await $fetch(`${config.public.apiBase}/blog/posts/${id}`, {
    method: 'DELETE'
  })
  data.value = data.value.filter(post => post.id !== id)
}

function getDropdownActions(post: Post) {
  return [
    [
      {
        label: 'Редагувати',
        icon: 'i-lucide-edit',
        onSelect: () => router.push(`/posts/${post.id}/edit`)
      },
      {
        label: 'Видалити',
        icon: 'i-lucide-trash',
        color: 'error',
        onSelect: () => deletePost(post.id)
      }
    ]
  ]
}

function formatDate(dateString: string | null): string {
  if (!dateString) return 'Неопубліковано'
  const date = new Date(dateString)
  return isNaN(date.getTime())
      ? 'Неопубліковано'
      : new Intl.DateTimeFormat('uk-UA', {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
      }).format(date)
}

const columns: TableColumn<Post>[] = [
  { accessorKey: 'id', header: '#' },
  { accessorKey: 'user.name', header: 'Автор' },
  { accessorKey: 'category.title', header: 'Категорія' },
  {
    accessorKey: 'title',
    header: 'Заголовок',
    cell: ({ row }) =>
        h(
            'a',
            {
              href: `/show/${row.original.id}`,
              class: 'text-blue-600 hover:underline'
            },
            row.original.title
        )
  },
  {
    accessorKey: 'published_at',
    header: 'Дата публікації',
    cell: ({ row }) => formatDate(row.original.published_at)
  },
  {
    accessorKey: 'actions',
    header: 'Дії',
    cell: () => null
  }
]
</script>


<template>
  <div class="w-full max-w-7xl mx-auto space-y-4 pb-8">
    <div class="my-6 px-4 flex justify-between items-center">
      <h1 class="text-4xl font-bold leading-tight text-gray-900">BlogPosts</h1>
      <UButton label="Додати пост" icon="i-heroicons-plus" @click="router.push('/posts/create')" />
    </div>

    <div class="overflow-x-auto">
      <UTable
          ref="table"
          :columns="columns"
          :data="data || []"
          :loading="loading"
          class="table-auto min-w-full text-left"
      >
        <template #actions-cell="{ row }">
          <UDropdownMenu :items="getDropdownActions(row.original)">
            <UButton
                icon="i-lucide-ellipsis-vertical"
                color="neutral"
                variant="ghost"
                aria-label="Дії"
            />
          </UDropdownMenu>
        </template>
      </UTable>
    </div>

    <div class="flex justify-center border-t border-gray-200 pt-4">
      <UPagination
          :page="pagination.pageIndex + 1"
          :total="total"
          :items-per-page="pagination.pageSize"
          :page-count="Math.ceil(total / pagination.pageSize)"
          @update:page="(p) => pagination.pageIndex = p - 1"
      />
    </div>
  </div>
</template>


