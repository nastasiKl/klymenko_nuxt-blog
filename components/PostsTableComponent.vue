<template>
  <div class="container">

    <div class="flex justify-center">

      <div class="w-full">

        <nav class="navbar bg-gray-100">

          <a href="/admin/blog/posts/create" class="">Додати</a>

        </nav>

        <div class="card">

          <div class="card-body">

            <table class="table table-auto">
      <thead>
      <tr>
        <th>#</th>
        <th>Автор</th>
        <th>Категорія</th>
        <th>Заголовок</th>
        <th>Дата публікації</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="post in posts">

        <td>{{ post.id }}</td>

        <td>{{ post.user.name }}</td>

        <td>{{ post.category.title }}</td>

        <td class="py-4 px-6 text-sm">
          <NuxtLink
              :to="`/posts/${post.id}`"
              class="font-semibold hover:underline transition-colors duration-200"
          >
            {{ post.title }}
          </NuxtLink>
        </td>

        <td>{{ post.published_at }}

        </td>
        <td class="py-4 px-6 text-sm">
                    <span
                        class="inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold"
                        :class="post.is_published
                        ? 'bg-green-100 text-green-800 border border-green-200'
                        : 'bg-yellow-100 text-yellow-800 border border-yellow-200'"
                    >
                      {{ post.is_published ? 'Опубліковано' : 'Чернетка' }}
                    </span>
        </td>
      </tr>
      </tbody>

            </table>

          </div>

        </div>

      </div>

    </div>

  </div>

</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
interface Post {
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

const posts = ref<Post[]>([]);
const loading = ref(false);
const config = useRuntimeConfig()

const getPosts = async () => {
  loading.value = true;
  try {
    let allPosts: Post[] = [];
    let currentPage = 1;
    let totalPages = 1;

    do {
      const apiUrl = `${config.public.apiBase}/blog/posts?page=${currentPage}&per_page=50`;
      console.log(`Завантажуємо сторінку ${currentPage} з ${apiUrl}`);

      const response = await $fetch<{
        data: Post[],
        meta: {
          current_page: number,
          last_page: number,
          total: number
        }
      }>(apiUrl);

      allPosts = [...allPosts, ...response.data];
      totalPages = response.meta.last_page;
      currentPage++;

      console.log(`Завантажено ${response.data.length} постів з сторінки ${response.meta.current_page}`);
      console.log(`Всього сторінок: ${totalPages}, поточна сторінка: ${response.meta.current_page}`);

    } while (currentPage <= totalPages);

    console.log(`Всього завантажено постів: ${allPosts.length}`);
    posts.value = allPosts;
  } catch (error) {
    console.error("Error fetching posts:", error);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  getPosts();
});
</script>
