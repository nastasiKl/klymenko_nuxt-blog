<script setup lang="ts">
import { getPaginationRowModel } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'
import { h } from 'vue'

const table = useTemplateRef('table')

type Post = {
  id: number;
  title: string;
  slug: string;
  is_published: boolean;
  published_at: string | null;
  user_id: number;
  category_id: number;
  user: { name: string };
  category: { title: string };
}

type PostResponse = {
  data: Post[];
  meta: {
    total: number;
    current_page: number;
    last_page: number;
    per_page: number;
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
  try {
    const response = await $fetch<PostResponse>(
        `${config.public.apiBase}/blog/posts?page=${currentPage.value}&per_page=${pageSize.value}`
    )
    total.value = response.meta.total
    data.value = response.data
  } finally {
    loading.value = false
  }
}

watch([currentPage, pageSize], fetchPosts, { immediate: true })

const columns: TableColumn<Post>[] = [
  { accessorKey: 'id', header: '#' },
  { accessorKey: 'user.name', header: 'Автор' },
  { accessorKey: 'category.title', header: 'Категорія' },
  {
    accessorKey: 'title',
    header: 'Заголовок',
    cell: ({ row }) => h(
        'a',
        { href: `/posts/${row.original.id}`, class: 'text-blue-600 hover:underline' },
        row.original.title
    )
  },
  {
    accessorKey: 'published_at',
    header: 'Дата публікації',
    cell: ({ row }) => {
      const date = row.original.published_at
      return date ? date.split('T')[0].split('-').reverse().join('.') : 'Неопубліковано'
    }
  }

]
</script>

<template>
  <div class="w-full max-w-screen-lg mx-auto space-y-4 pb-4">
    <div class="my-6 px-4">
      <h1 class="text-4xl font-bold leading-tight text-gray-900">
        BlogPosts
      </h1>
    </div>
    <UTable
        ref="table"
        :columns="columns"
        :data="data || []"
        :loading="loading"
        class="table-auto w-full text-left"
    />

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
