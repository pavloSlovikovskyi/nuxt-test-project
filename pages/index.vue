<template>
  <div class="p-6">
    <h1 class="text-2xl font-bold mb-4">Product List</h1>

    <!-- Інформація про статус -->
    <div class="mb-4 p-2 bg-gray-100 rounded">
      <p>Status: {{ pending ? 'Loading...' : error ? 'Error' : 'Loaded!' }}</p>
      <p>Amount of products: {{ rawProducts.length }}</p>
    </div>

    <!-- Пошук -->
    <div class="mb-6">
      <input
          v-model="search"
          type="text"
          class="w-full p-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Search..."
      />
    </div>

    <!-- Таблиця -->
    <div v-if="rawProducts.length > 0" class="border rounded overflow-x-auto">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
        <tr>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Photo</th>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Name</th>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Description</th>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Price</th>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Rating</th>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Category</th>
          <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Brand</th> <!-- Нова колонка -->
        </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
        <tr v-for="product in paginatedRows" :key="product.id" class="hover:bg-gray-50">
          <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
            <img
                :src="product.thumbnail || getFirstImage(product)"
                @error="handleImageError"
                alt="Product Image"
                class="w-24 h-24 object-contain"
            />
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{{ product.title }}</td>
          <td class="px-6 py-4 whitespace-normal text-sm text-gray-500 max-w-md truncate">{{ product.description }}</td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${{ product.price }}</td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
        <span :class="product.rating < 4.5 ? 'text-red-500' : 'text-green-500'">
          {{ product.rating }}
        </span>
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ product.category }}</td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ product.brand }}</td>
        </tr>
        </tbody>
      </table>

    </div>

    <!-- Пагінація -->
    <div v-if="paginatedRows.length > 0" class="mt-4 flex justify-between items-center">
      <button
          @click="page = Math.max(page - 1, 1)"
          class="px-4 py-2 text-sm font-medium text-white bg-blue-500 rounded hover:bg-blue-600"
          :disabled="page === 1"
      >
        Back
      </button>
      <span>Page {{ page }} out of {{ totalPages }}</span>
      <button
          @click="page = Math.min(page + 1, totalPages)"
          class="px-4 py-2 text-sm font-medium text-white bg-blue-500 rounded hover:bg-blue-600"
          :disabled="page === totalPages"
      >
        Next
      </button>
    </div>
  </div>
</template>

<script setup>
// Завантаження даних з API
const { data: productsData, pending, error } = await useFetch('https://dummyjson.com/products')

// Просте зберігання даних
const rawProducts = ref([])

// Пошук
const search = ref('')

// Пагінація
const page = ref(1)
const pageCount = 5

onMounted(async () => {
  // Чекаємо поки дані завантажуються
  while (pending.value) {
    await new Promise(resolve => setTimeout(resolve, 100))
  }

  // Якщо дані успішно прийшли
  if (productsData.value && productsData.value.products) {
    // Беремо масив продуктів
    rawProducts.value = productsData.value.products
  }
})

// Фільтрація
const filteredRows = computed(() =>
    rawProducts.value.filter(row =>
        row.title.toLowerCase().includes(search.value.toLowerCase()) ||
        row.description.toLowerCase().includes(search.value.toLowerCase()) ||
        row.category.toLowerCase().includes(search.value.toLowerCase())
    )
)

// Пагіновані рядки
const totalPages = computed(() => Math.ceil(filteredRows.value.length / pageCount))

const paginatedRows = computed(() => {
  const start = (page.value - 1) * pageCount
  return filteredRows.value.slice(start, start + pageCount)
})
</script>
