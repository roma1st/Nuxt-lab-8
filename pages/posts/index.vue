<template>
  <div class="flex flex-col">
    <UNotifications />
    <div class="me-5">
      <NuxtLink to="/posts/create">
         <button class="bg-transparent hover:bg-green-500 float-end text-green-700 font-semibold hover:text-white py-1 px-2 border border-green-500 hover:border-transparent rounded">
          Add Post
        </button>
      </NuxtLink>
    </div>
    <div>
      <UTable class="mt-3" :columns="columns" :rows="rows" :total="totalPosts">
        <template #actions-data="{ row }">
          <UDropdown :items="items(row)">
            <UButton color="gray" variant="ghost" icon="i-heroicons-ellipsis-horizontal-20-solid" />
          </UDropdown>
        </template>
      </UTable>
      <div class="d-flex mt-5">
        <UPagination v-model="page" :page-count="5" :total="totalPosts" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from 'axios';
import { ref, computed, watch } from 'vue';
import { API_SERVER_URL } from '../../untils/constants';

const toast = useToast();

interface Post {
  id: number;
  title: string;
  published_at: string;
  user: User;
  category: Category;
}

interface User {
  id: number;
  name: string;
}

interface Category {
  id: number;
  title: string;
}

const columns = [
  { key: 'id', label: 'ID' },
  { key: 'title', label: 'Title' },
  { key: 'published_at', label: 'Published At' },
  { key: 'user', label: 'User' },
  { key: 'category', label: 'Category' },
  { key: 'actions', label: 'Actions' }
];

const page = ref(1);
const posts = ref<Post[]>([]);
const totalPosts = ref(0);

const getPosts = async (page = 1) => {
  try {
    const response = await axios.get(`${API_SERVER_URL}/api/blog/posts?page=${page}`);
    posts.value = response.data.data; // Assuming response structure has 'data' key containing posts
    totalPosts.value = response.data.total;
    console.log(response.data);
  } catch (error) {
    console.error('Error fetching posts:', error);
  }
};

watch(page, () => {
  getPosts(page.value);
});

getPosts(page.value);

const rows = computed(() => {
  if (posts.value && posts.value.length > 0) {
    return posts.value.map(post => ({
      id: post.id,
      title: post.title,
      published_at: post.published_at,
      user: post.user ? post.user.name : 'N/A',
      category: post.category ? post.category.title : 'N/A'
    }));
  } else {
    return [];
  }
});

const items = (post: Post) => [
  [
    {
      label: 'Edit',
      icon: 'i-heroicons-pencil-square-20-solid',
      click: () => window.location.href = `/posts/edit/${post.id}`
    },
    {
      label: 'Details',
      icon: 'i-heroicons-information-circle-20-solid',
      click: () => window.location.href = `/posts/${post.id}`
    }
  ],
  [
    {
      label: 'Delete',
      icon: 'i-heroicons-trash-20-solid',
      click: () => {
        axios.delete(`${API_SERVER_URL}/api/blog/posts/delete/${post.id}`)
            .then(res => {
              console.log(res);
              toast.add({ title: 'Success', description: 'Post was deleted' });
              getPosts(page.value);
            })
            .catch(error => {
              console.error(error);
              alert('Failed to delete post!');
            });
      }
    }
  ]
];
</script>