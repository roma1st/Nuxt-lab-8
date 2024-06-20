<template>
  <div>
    <div v-if="category">
      <div class="flex center mb-4"></div>
      <div class="flex justify-center">
        <div class="w-full md:w-2/3">
          <div class="bg-gray-800 shadow-md rounded-lg p-4">
            <h3 class="text-xl font-bold mb-4">Category Detail</h3>
            <div class="form-group mb-4">
              <label for="title" class="block text-gray-100">Title</label>
              <input type="text" class="w-full p-2 border rounded" id="title" v-model="category.title" readonly>
            </div>
            <div class="form-group mb-4" v-if="category.parent_category">
              <label for="category" class="block text-gray-100">Parent category</label>
              <input type="text" class="w-full p-2 border rounded" id="category" v-model="category.parent_category.title" readonly>
            </div>
            <div class="form-group mb-4">
              <label for="description" class="block text-gray-100">Description</label>
              <input type="text" class="w-full p-2 border rounded" id="description" v-model="category.description" readonly>
            </div>
            <div class="form-group mb-4">
              <h3 class="text-xl font-bold mb-4">Slug</h3>
              <textarea class="w-full p-2 border rounded" id="slug" v-model="category.slug" readonly rows="5"></textarea>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      Loading...
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

const route = useRoute();
const id = Number(route.params.id);

interface Category {
  id: number;
  title: string;
  description: string;
  parent_category?: ParentCategory; // Parent category can be optional
  slug: string;
}

interface ParentCategory {
  id: number;
  title: string;
}

const category = ref<Category>();

const getCategory = async (id: number) => {
  try {
    const response = await $fetch(`http://localhost:8000/api/blog/categories/${id}`);
    category.value = response;
  } catch (error) {
    console.error('Failed to fetch category:', error);
  }
};

onMounted(() => {
  getCategory(id);
});
</script>
