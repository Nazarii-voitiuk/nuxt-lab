<template>
  <div class="container">
    <h1 class="title">📦 Список продуктів</h1>

    <!-- Пошук -->
    <input
        v-model="searchQuery"
        type="text"
        placeholder="🔍 Пошук за назвою, брендом, категорією, описом, ціною або оцінкою..."
        class="search-input"
    />

    <!-- Таблиця -->
    <table class="product-table">
      <thead>
      <tr>
        <th @click="sortBy('title')">Назва</th>
        <th @click="sortBy('description')">Опис</th>
        <th @click="sortBy('price')">Ціна</th>
        <th @click="sortBy('rating')">Оцінка</th>
        <th @click="sortBy('brand')">Бренд</th>
        <th @click="sortBy('category')">Категорія</th>
        <th>Фото</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="product in paginatedProducts" :key="product.id">
        <td>{{ product.title }}</td>
        <td>{{ product.description }}</td>
        <td>{{ product.price }}$</td>
        <td :class="product.rating < 4.5 ? 'low-rating' : 'high-rating'">
          {{ product.rating }}
        </td>
        <td>{{ product.brand }}</td>
        <td>{{ product.category }}</td>
        <td>
          <img :src="product.thumbnail" alt="product" class="product-image" />
        </td>
      </tr>
      </tbody>
    </table>

    <!-- Пагінація -->
    <div class="pagination">
      <button @click="prevPage" :disabled="page === 1">⬅ Назад</button>
      <span>Сторінка {{ page }} з {{ totalPages }}</span>
      <button @click="nextPage" :disabled="page === totalPages">➡ Далі</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const products = ref([])
const searchQuery = ref('')
const sortKey = ref('')
const sortAsc = ref(true)
const page = ref(1)
const perPage = 5

onMounted(async () => {
  const res = await fetch('https://dummyjson.com/products')
  const json = await res.json()
  products.value = json.products
})

// Нормалізація тексту для пошуку
const normalize = (str) => {
  return String(str)
      .toLowerCase()
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, '')
      .replace(/[^\w\s.]/gi, '')
      .trim()
}

// Фільтровані продукти з повним пошуком
const filteredProducts = computed(() => {
  const query = normalize(searchQuery.value)
  if (!query) return products.value

  const terms = query.split(/\s+/)

  return products.value.filter(product => {
    const fields = [
      product.title,
      product.description,
      product.brand,
      product.category,
      String(product.price),
      String(product.rating)
    ].map(normalize)

    return terms.every(term =>
        fields.some(field => field.includes(term))
    )
  })
})

// Сортовані продукти (усі, до пагінації)
const sortedProducts = computed(() => {
  const items = [...filteredProducts.value]
  if (!sortKey.value) return items

  return items.sort((a, b) => {
    const aVal = a[sortKey.value]
    const bVal = b[sortKey.value]
    if (typeof aVal === 'number' && typeof bVal === 'number') {
      return sortAsc.value ? aVal - bVal : bVal - aVal
    } else {
      return sortAsc.value
          ? String(aVal).localeCompare(String(bVal))
          : String(bVal).localeCompare(String(aVal))
    }
  })
})

// Скидання сторінки при зміні фільтру чи сортування
watch([searchQuery, sortKey, sortAsc], () => {
  page.value = 1
})

const totalPages = computed(() =>
    Math.ceil(sortedProducts.value.length / perPage)
)

const paginatedProducts = computed(() => {
  const start = (page.value - 1) * perPage
  return sortedProducts.value.slice(start, start + perPage)
})

const nextPage = () => {
  if (page.value < totalPages.value) page.value++
}
const prevPage = () => {
  if (page.value > 1) page.value--
}

const sortBy = (key) => {
  if (sortKey.value === key) {
    sortAsc.value = !sortAsc.value
  } else {
    sortKey.value = key
    sortAsc.value = true
  }
}
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.title {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 1.5rem;
}

.search-input {
  width: 100%;
  padding: 10px;
  font-size: 1rem;
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  background-color: #f9f9f9;
}

.product-table th,
.product-table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: center;
}

.product-table th {
  background-color: #e2e8f0;
  cursor: pointer;
  user-select: none;
}

.product-table tbody tr:hover {
  background-color: #f0f8ff;
  transition: 0.2s;
}

.low-rating {
  color: red;
  font-weight: bold;
}

.high-rating {
  color: green;
  font-weight: bold;
}

.product-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 4px;
}

.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
}

.pagination button {
  padding: 8px 16px;
  border: none;
  background-color: #0070f3;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.pagination button:disabled {
  background-color: #aaa;
  cursor: not-allowed;
}
</style>
