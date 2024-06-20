<template>
  <div class="container">
    <nav class="navbar bg-gray-800">
      <a href="/admin/blog/posts/create" class="btn btn-primary">Додати</a>
    </nav>

    <div class="card">
      <div class="card-body">
        <input
            type="text"
            v-model="searchQuery"
            placeholder="Пошук за заголовком"
            class="search-input"
        />
        <div class="table-responsive">
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
            <tr v-for="post in filteredPaginatedPosts" :key="post.id">
              <td>{{ post.id }}</td>
              <td>{{ post.user.name }}</td>
              <td>{{ post.category.title }}</td>
              <td>
                <a :href="'/admin/blog/posts/' + post.id + '/edit'">
                  {{ post.title }}
                </a>
              </td>
              <td>{{ post.published_at }}</td>
            </tr>
            </tbody>
          </table>
        </div>

        <nav class="pagination">
          <button :disabled="page.value === 1" @click="previousPage" class="pagination-button">
            Попередня
          </button>
          <button
            v-for="p in pageCount"
            :key="p"
            @click="changePage(p)"
            :class="{ 'active': page.value === p }"
            class="pagination-button"
          >
            {{ p }}
          </button>
          <button :disabled="page.value === pageCount" @click="nextPage" class="pagination-button">
            Наступна
          </button>
        </nav>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';

const posts = ref<any[]>([]);
const page = ref(1); // Поточна сторінка
const perPage = 5; // Кількість постів на сторінці
const searchQuery = ref(''); // Строка для пошуку

// Функція для фільтрації записів за пошуковим запитом
const filteredPosts = computed(() =>
    posts.value.filter(post =>
        post.title.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
);

// Обчислення кількості сторінок
const pageCount = computed(() => Math.ceil(filteredPosts.value.length / perPage));

// Обчислення постів для поточної сторінки
const filteredPaginatedPosts = computed(() => {
  const start = (page.value - 1) * perPage;
  const end = start + perPage;
  return filteredPosts.value.slice(start, end);
});

// Функції для перемикання сторінок
const previousPage = () => {
  if (page.value > 1) {
    page.value--;
  }
};

const nextPage = () => {
  if (page.value < pageCount.value) {
    page.value++;
  }
};

// Функція для зміни сторінки за номером
const changePage = (pageNumber: number) => {
  page.value = pageNumber;
};

// Функція для завантаження постів
const getPosts = () => {
  fetch('http://127.0.0.1:8000/api/blog/posts')
      .then(response => {
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        return response.json();
      })
      .then(data => {
        posts.value = data;
      })
      .catch(error => {
        console.error('There has been a problem with your fetch operation:', error);
      });
};

// Виклик функції для завантаження постів
getPosts();
</script>

<style scoped>
.container {
  width: 100%; /* Замість max-width */
  margin: 0 auto;
  padding: 0 20px;
  box-sizing: border-box;
}

.navbar {
  width: 100%;
  margin-bottom: 20px;
  padding: 10px 20px;
  box-sizing: border-box;
  background-color: #1a1a1a;
  border-radius: 8px;
  display: flex;
  justify-content: flex-end;
}

.card {
  width: 100%;
  margin: 0 auto;
  background-color: #2a2a2a;
  border-radius: 8px;
}

.card-body {
  padding: 20px;
}

.search-input {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #444;
  border-radius: 4px;
  background-color: #3a3a3a;
  color: white;
  font-size: 16px;
}

.table-responsive {
  overflow-x: auto;
}

.table {
  width: 100%; /* Ширина на всю сторінку */
  border-collapse: collapse;
  background-color: #3a3a3a;
  box-sizing: border-box;
  border-radius: 8px;
  overflow: hidden;
}

.table th,
.table td {
  padding: 12px 10px;
  border-bottom: 1px solid #555;
  text-align: left;
  color: white;
}

.table th {
  background-color: #4a4a4a;
  font-weight: bold;
}

.table td a {
  color: #66bfff;
  text-decoration: none;
}

.table td a:hover {
  text-decoration: underline;
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.pagination-button {
  margin: 0 5px;
  padding: 8px 16px;
  border: 1px solid #444;
  background-color: #333;
  color: white;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.2s;
}

.pagination-button:hover {
  background-color: #555;
}

.pagination-button.active {
  background-color: #007bff;
  color: white;
}

.pagination-button:disabled {
  background-color: #444;
  cursor: not-allowed;
}
</style>
