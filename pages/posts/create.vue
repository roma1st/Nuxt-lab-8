<template>
  <div>
    <div v-if="post">
      <div class="flex center mb-4"></div>
      <UForm :schema="schema" :state="state" @submit="submitPost">
        <div class="flex justify-center">
          <div class="w-full md:w-2/3">
            <div class="bg-gray-800 shadow-md rounded-lg p-4">
              <h3 class="text-xl font-bold mb-4">Додати</h3>
              <UFormGroup label="Title" name="title">
                <UInput class="w-full" id="title" v-model="state.title" />
              </UFormGroup>
              <UFormGroup label="Category" name="category">
                <select class="w-full p-2 bg-gray-900 text-white" id="category" v-model="state.category_id">
                  <option v-for="category in categories" :key="category.id" :value="category.id">
                    {{ category.title }}
                  </option>
                </select>
              </UFormGroup>
              <UFormGroup label="Post Content" name="content_raw">
                <UTextarea class="w-full" id="content" v-model="state.content_raw" :rows="5" />
              </UFormGroup>
              <UFormGroup label="Excerpt" name="excerpt">
                <UInput class="w-full" id="excerpt" v-model="state.excerpt" />
              </UFormGroup>
              <UFormGroup label="Publish" name="is_published">
                <input type="checkbox" id="is_published" v-model="state.is_published" />
              </UFormGroup>
              <UFormGroup label="Slug" name="slug">
                <UInput class="w-full" id="slug" v-model="state.slug" />
              </UFormGroup>

              <!-- Submit Button -->
              <UButton type="submit" class="bg-blue-500 text-white px-4 py-2 rounded-md mt-4">
                Відправити
              </UButton>
            </div>
          </div>
        </div>
      </UForm>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from 'vue';
import { z } from 'zod';
import { useRouter } from 'vue-router';
import { API_SERVER_URL } from '../../untils/constants';

interface Category {
  id: number;
  title: string;
}

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
  slug: '',
});

const schema = z.object({
  title: z.string().min(5, 'Must be at least 5 characters'),
  content_raw: z.string().min(5, 'Must be at least 5 characters'),
  category_id: z.number(),
  is_published: z.boolean(),
  excerpt: z.string(),
  slug: z.string(),
});

const post = ref<Post>({
  title: '',
  content_raw: '',
  category_id: 0,
  is_published: false,
  excerpt: '',
  slug: '',
});

const tabs = ref(['Details', 'Content']);
const activeTab = ref('Details');
const router = useRouter();
const categories = ref<Category[]>([]);

const getCategories = async () => {
  try {
    const response = await $fetch<Category[]>(`${API_SERVER_URL}/api/blog/categories/forCombobox`);
    categories.value = response;
  } catch (error) {
    console.error('Failed to fetch categories:', error);
  }
};

onMounted(() => {
  getCategories();
});

const submitPost = async () => {
  try {
    const validatedData = schema.parse(state);
    console.log(validatedData);
    await $fetch(`${API_SERVER_URL}/api/blog/posts/create`, {
      method: 'POST',
      body: validatedData,
    });
    router.push('/posts/');
  } catch (error) {
    console.error('Failed to submit post:', error);
  }
};
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
