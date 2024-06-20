<template>
  <div v-if="post" class="container mx-auto mt-8">
    <div class="flex justify-center mb-4">
      <div class="w-full md:w-2/3">
        <div class="bg-gray-800 shadow-md rounded-lg p-4">
          <h3 class="text-2xl font-bold mb-4 text-white">Post Detail</h3>
          <div class="my-4">
              <div v-if="post.is_published" class="text-green-500 mb-4">
                Published on {{ formatDate(post.published_at) }}
              </div>
              <div v-else class="text-red-500 mb-4">
                Draft
              </div>
              <div class="form-group mb-4">
                <label for="title" class="block text-gray-100">Title</label>
                <input type="text" class="w-full p-2 border rounded bg-gray-700 text-gray-200" id="title" v-model="post.title" readonly>
              </div>
              <div class="form-group mb-4">
                <label for="category" class="block text-gray-100">Category</label>
                <input type="text" class="w-full p-2 border rounded bg-gray-700 text-gray-200" id="category" v-model="post.category.title" readonly>
              </div>
              <div class="form-group mb-4">
                <label for="user" class="block text-gray-100">User</label>
                <input type="text" class="w-full p-2 border rounded bg-gray-700 text-gray-200" id="user" v-model="post.user.name" readonly>
              </div>
              <div class="form-group mb-4">
                <h3 class="text-xl font-bold mb-2 text-white">Post Content</h3>
                <textarea class="w-full p-2 border rounded bg-gray-700 text-gray-200" id="content" v-model="post.content_raw" readonly rows="5"></textarea>
              </div>
              <div>
                <div class="text-green-500 mb-4">Created at {{ formatDate(post.created_at) }}</div>
                <div class="text-yellow-500 mb-4">Updated at {{ formatDate(post.updated_at) }}</div>
                <div class="form-group mb-4">
                  <label for="excerpt" class="block text-gray-100">Excerpt</label>
                  <input type="text" class="w-full p-2 border rounded bg-gray-700 text-gray-200" id="excerpt" v-model="post.excerpt" readonly>
                </div>
                <div class="form-group mb-4">
                  <label for="slug" class="block text-gray-100">Slug</label>
                  <input type="text" class="w-full p-2 border rounded bg-gray-700 text-gray-200" id="slug" v-model="post.slug" readonly>
                </div>
              </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-else class="flex justify-center items-center h-screen text-gray-600">Loading...</div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useRoute } from 'vue-router';
import { format } from 'date-fns';

const route = useRoute();
const id = Number(route.params.id);

interface Post {
  id: number;
  title: string;
  content_raw: string;
  published_at: string;
  user: User;
  category: Category;
  is_published: boolean;
  excerpt: string;
  slug: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
}

interface User {
  name: string;
}

interface Category {
  title: string;
}

const post = ref<Post>();
const tabs = ref(['Details', 'Content']);
const activeTab = ref('Details');

const getPost = async (id: number) => {
  const response = await $fetch<Post>(`http://localhost:8000/api/blog/posts/${id}`);
  post.value = response;
};

const formatDate = (dateString: string) => {
  return format(new Date(dateString), 'dd-MM-yyyy HH:mm');
};

getPost(id);
</script>

<style scoped>
/* Add your custom scoped styles here */
.container {
  max-width: 800px;
}

.form-group input,
.form-group textarea {
  background-color: #2d2d2d; /* Dark gray background */
  color: #e5e7eb; /* Light gray text */
}

.form-group label {
  color: #9ca3af; /* Gray label text */
}

.button {
  cursor: pointer;
}
</style>
