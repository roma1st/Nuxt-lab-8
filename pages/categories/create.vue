<template>
  <UForm :schema="schema" :state="state" @submit='submitPost'>
    <div class="flex center mb-4"></div>
    <div class="flex justify-center">
      <div class="w-full md:w-2/3">
        <div class="bg-gray-800 shadow-md rounded-lg p-4">
          <h3 class="text-xl font-bold mb-4">Category Detail</h3>
          <UFormGroup label="Title" name="title">
            <UInput type="text" class="w-full " id="title" v-model="state.title" />
          </UFormGroup>
          <UFormGroup label="Parent category" name="category">
            <select class="w-full bg-gray-900 p-1 rounded-md" id="category" v-model="state.parent_id">
              <option v-for="category in pcategories" :key="category.id" :value="category.id">
                {{ category.title }}
              </option>
            </select>
            </UFormGroup>
          <UFormGroup label="Slug" name="slug">
            <UInput type="text" class="w-full " id="slug" v-model="state.slug" />
          </UFormGroup>
          <UFormGroup label="description" name="description">
            <UTextarea class="w-full " id="description" v-model="state.description" :rows="5"></UTextarea>
          </UFormGroup>
          <UButton type="submit">Відправити</UButton>
        </div>
      </div>
    </div>
  </UForm>

</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import {useRoute, useRouter} from 'vue-router';
import {z} from "zod";
import {API_SERVER_URL} from "~/untils/constants";
const router = useRouter();

const route = useRoute();
const id = Number(route.params.id);

interface Category {
  title: string;
  description: string;
  parent_category: ParentCategory;
  slug: string;
  parent_id: number;
}

interface ParentCategory {
  id: number;
  title: string;
}

const state = reactive({
  title: '',
  description: '',
  parent_category: { id : 0, title: '' },
  slug: '',
  parent_id : 0
})
const schema = z.object({
  title: z.string().min(5,'Must be at least 3 characters'),
  description: z.string().min(3,'Must be at least 5 characters'),
  parent_id: z.number().optional(),
  slug: z.string()
})
const category = ref<Category>({
  title: '',
  description: '',
  parent_category: { id : 0, title: '' },
  slug: '',
  parent_id : 0
});
const pcategories = ref<Category[]>([]);

const getCategories = async () => {
  try {
    const response = await $fetch(`${API_SERVER_URL}/api/blog/categories/forCombobox`);
    pcategories.value = response;
  } catch (error) {
    console.error('Failed to fetch categories:', error);
  }
};
onMounted(() => {
  getCategories();
});
const submitPost = async () => {
  try {
    console.log(state)
    const validatedData = schema.parse(state);
    await $fetch(`${API_SERVER_URL}/api/blog/categories/create`, {
      method: 'POST',
      body:  validatedData
    });
    router.push('/categories/');
  } catch (error) {
    console.error('Failed to submit post:', error);
  }
};
</script>
