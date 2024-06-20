<template>
  <div>
    <div class="flex center mb-4"></div>

    <UForm :schema="schema" :state="state" @submit="submitPost">
      <div class="flex justify-center">
        <div class="w-full md:w-2/3">
          <div class="bg-gray-800 shadow-md rounded-lg p-4">
            <h3 class="text-xl font-bold mb-4">Edit Post</h3>
            <UFormGroup label="Title" name="title">
              <UInput class="w-full p-2 border rounded" id="title" v-model="state.title" />
            </UFormGroup>
            <UFormGroup label="Category" name="category">
              <select class="w-full p-2 border rounded" id="category" v-model="state.category_id">
                <option v-for="category in categories" :key="category.id" :value="category.id">
                  {{ category.title }}
                </option>
              </select>
            </UFormGroup>
            <UFormGroup label="Post Content" name="content_raw">
              <textarea class="w-full p-2 border rounded" id="content" v-model="state.content_raw" rows="5"></textarea>
            </UFormGroup>
            <UFormGroup label="Excerpt" name="excerpt">
              <UInput class="w-full p-2 border rounded" id="excerpt" v-model="state.excerpt" />
            </UFormGroup>
            <UFormGroup label="Publish" name="is_published">
              <input type="checkbox" id="is_published" v-model="state.is_published" />
            </UFormGroup>
            <UFormGroup label="Slug" name="slug">
              <UInput class="w-full p-2 border rounded" id="slug" v-model="state.slug" />
            </UFormGroup>
            <button type="submit" class="btn-submit">Відправити</button>
          </div>
        </div>
      </div>
    </UForm>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { z } from 'zod';
import { API_SERVER_URL } from '../../../untils/constants';

interface Post {
  id?: number;
  title: string;
  content_raw: string;
  category_id: number;
  is_published: boolean;
  excerpt: string;
  slug: string;
}

const state = reactive<Post>({
  title: '',
  content_raw: '',
  category_id: 0,
  is_published: false,
  excerpt: '',
  slug: ''
});

const schema = z.object({
  title: z.string().min(3, 'Must be at least 3 characters'),
  content_raw: z.string().min(5, 'Must be at least 5 characters'),
  category_id: z.number().min(1, 'Category is required'),
  is_published: z.boolean(),
  excerpt: z.string().min(5, 'Excerpt must be at least 5 characters'),
  slug: z.string().min(1, 'Slug is required')
});

const tabs = ref(['Details', 'Content']);
const activeTab = ref('Details');
const router = useRouter();
const route = useRoute();
const categories = ref([]);

const backToIndex = () => {
  router.push('/posts/');
};

const getCategories = async () => {
  try {
    const response = await $fetch(`${API_SERVER_URL}/api/blog/categories/forCombobox`);
    categories.value = response;
  } catch (error) {
    console.error('Failed to fetch categories:', error);
  }
};

const fetchPost = async () => {
  try {
    const response = await $fetch(`${API_SERVER_URL}/api/blog/posts/${route.params.id}`);
    const postData = response;
    Object.assign(state, {
      title: postData.title,
      content_raw: postData.content_raw,
      category_id: postData.category_id,
      is_published: !!postData.is_published,
      excerpt: postData.excerpt,
      slug: postData.slug
    });

    if (categories.value.length > 0) {
      const selectedCategory: any = categories.value.find((cat: any) => cat.id === postData.category_id);
      if (selectedCategory) {
        state.category_id = selectedCategory.id;
      }
    }
  } catch (error) {
    console.error('Failed to fetch post:', error);
  }
};

const submitPost = async () => {
  try {
    const validatedData = schema.parse(state);

    const postData = {
      ...validatedData,
      category_id: validatedData.category_id,
      is_published: validatedData.is_published,
      id: route.params.id
    };

    await $fetch(`${API_SERVER_URL}/api/blog/posts/edit/${route.params.id}`, {
      method: 'PUT',
      body: postData
    });

    router.push('/posts/');
  } catch (error) {
    console.error('Failed to submit post:', error);
  }
};

onMounted(async () => {
  await getCategories();
  await fetchPost();
});
</script>

<style scoped>
.btn-submit {
  margin-top: 10px;
  padding: 10px 20px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-submit:hover {
  background-color: #0056b3;
}
</style>
