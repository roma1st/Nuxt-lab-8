<template>
  <div class="flex flex-col">
    <UNotifications />
    <div class="me-5">
      <NuxtLink to="/categories/create">
        <button class="bg-transparent hover:bg-green-500 float-end text-green-700 font-semibold hover:text-white py-1 px-2 border border-green-500 hover:border-transparent rounded">
          Add Category
        </button>
      </NuxtLink>
    </div>
    <div>
      <UTable class="mt-3" :columns="columns" :rows="rows" :total="totalCategories">
        <template #actions-data="{ row }">
          <UDropdown :items="items(row)">
            <UButton color="gray" variant="ghost" icon="i-heroicons-ellipsis-horizontal-20-solid" />
          </UDropdown>
        </template>
      </UTable>
      <div class="d-flex mt-5">
        <UPagination v-model="page" :page-count="5" :total="totalCategories" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { API_SERVER_URL } from '../../untils/constants';  
import { ref, computed, watch } from 'vue';

interface Category {
  id: number;
  title: string;
  parent_category: Category | null;
  parent_id: number;
}
const toast = useToast()

const columns = [
  { key: 'id', label: '#' },
  { key: 'title', label: 'Назва' },
  { key: 'parent_category_title', label: 'Батьківська категорія' },
  { key: 'actions', label: 'Дії' }
];

const page = ref(1);
const totalPages = ref(1);
const categories = ref<Category[]>([]);
const totalCategories = ref(0);
console.log(totalPages.value);
const getCategories = async (page = 1) => {
  try {
    const response = await await $fetch(`${API_SERVER_URL}/api/blog/categories?page=${page}`);
    categories.value = response.data;
    totalPages.value = response.last_page;
    totalCategories.value = response.total;
    console.log(response)
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};

watch(page, () => {
  getCategories(page.value);
});

getCategories();

const rows = computed(() => {
  return categories.value.map(category => ({
    id: category.id,
    title: category.title,
    parent_category_title: category.parent_category ? category.parent_category.title : 'N/A'
  }));
});

const items = (category: Category) => [
  [
    {
      label: 'Редагувати',
      icon: 'i-heroicons-pencil-square-20-solid',
      click: () => window.location.href = (`/categories/edit/${category.id}`)
    },
    {
      label: 'Деталі',
      icon: 'i-heroicons-information-circle-20-solid',
      click: () => window.location.href = (`/categories/${category.id}`)
    }
  ],
  [
    {
      label: 'Видалити',
      icon: 'i-heroicons-trash-20-solid',
      click: () => {
          $fetch(`${API_SERVER_URL}/api/blog/categories/delete/${category.id}`, {
            method: 'DELETE',
            body: { id: category.id }
          })
              .then(res => {
                console.log(res);
                toast.add({ title: 'Success',description:"Category was deleted" })
                getCategories(page.value);
              })
              .catch(error => {
                console.error(error);
                alert('Не вдалось видалити категорію!');
              });
      }
    }
  ]
];
</script>