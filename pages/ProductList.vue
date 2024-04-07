<script setup lang="ts">
import { ref, computed } from 'vue';
const { pending,  data } = await useLazyAsyncData<any>('products',()=> $fetch('https://dummyjson.com/products') as any);
useHead({
  title: "Products"
})
const products = data.value.products;
const columns = [{
  key: 'id',
  label: 'ID',
  sortable: true
}, {
  key: 'title',
  label: 'Title',
  sortable: true
}, {
  key: 'description',
  label: 'Description',
  sortable: true
}, {
  key: 'price',
  label: 'Price',
  sortable: true
}, {
  key: 'rating',
  label: 'Rating',
  sortable: true,
  slot: 'rating-data'
}, {
  key: 'brand',
  label: 'Brand',
  sortable: true
}, {
  key: 'category',
  label: 'Category',
  sortable: true
}, {
  key: 'thumbnail',
  label: 'Thumbnail',
  sortable: true,
  slot: 'thumbnail-data'
}]

const q = ref('')
const page = ref(1)
const pageCount = 5

const sort = ref({ column: 'id', direction: 'asc' })

const filteredRows = computed(() => {
  if (!q.value) {
    return products
  }

  return products.filter((product) => {
    return Object.values(product).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase())
    })
  })
})

const sortedRows = computed(() => {
  const sortedProducts = [...filteredRows.value]
  const { column, direction } = sort.value

  if (column && direction) {
    sortedProducts.sort((a, b) => {
      const aValue = a[column]
      const bValue = b[column]
      if (aValue < bValue) return direction === 'asc' ? -1 : 1
      if (aValue > bValue) return direction === 'asc' ? 1 : -1
      return 0
    })
  }

  return sortedProducts
})

const paginatedRows = computed(() => {
  const startIndex = (page.value - 1) * pageCount
  return sortedRows.value.slice(startIndex, startIndex + pageCount)
})
</script>

<template>
  <div>
    <div class="flex px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput v-model="q" placeholder="Filter products..." />
    </div>

    <UTable :rows="paginatedRows" :columns="columns" :sort="sort">
      <template #rating-data="{ row }"> <!-- Додано шаблон rating-data -->
        <span :style="{ color: row.rating > 4.5 ? 'green' : 'red' }">
          {{ row.rating }}
        </span>
      </template>
      <template #thumbnail-data="{ row }"> <!-- Додано шаблон thumbnail-data -->
        <img class="w-[100px] h-[100px]" :src="row.thumbnail" alt="Thumbnail" />
      </template>
    </UTable>

    <div class="flex justify-end px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length" />
    </div>
  </div>
</template>
