<template>
  <div class="container mx-auto">
    <div class="flex flex-col gap-6">
      <div class="flex flex-col gap-4">
        <h1 class="text-3xl font-bold text-slate-800">Product Management</h1>

        <!-- Search and Filter Bar -->
        <div class="flex flex-col md:flex-row gap-4 items-start md:items-center justify-between">
          <div class="relative w-full md:w-1/3">
            <Search class="absolute left-3 top-1/2 transform -translate-y-1/2 text-slate-400" :size="18" />
            <input v-model="searchTerm" placeholder="Search products..."
              class="w-full px-6 py-3 bg-white rounded-lg border border-slate-200 focus:outline-none focus:ring-2 focus:ring-slate-200 pl-10" />
          </div>

          <div class="flex flex-wrap gap-2 items-center">
            <!-- Filters Button -->
            <button @click="isFilterSheetOpen = true"
              class="inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
              <Filter :size="16" />
              <span>Filters</span>
            </button>

            <!-- Sort Button -->
            <button @click="toggleSortDirection"
              class="inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
              <SlidersHorizontal :size="16" />
              <span>Sort: {{ sortByLabel }}</span>
              <component :is="sortDirection === 'asc' ? ChevronUp : ChevronDown" :size="16" />
            </button>

            <!-- CSV Upload Button -->
            <div class="relative">
              <input type="file" ref="fileInput" accept=".csv,.xlsx,.xls" @change="handleFileUpload"
                class="absolute inset-0 w-full h-full opacity-0 cursor-pointer" />
              <button
                class="inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
                <Upload :size="16" />
                <span>Import CSV/Excel</span>
              </button>
            </div>

            <!-- card view / table view -->
            <div class="relative">
              <Button @click="isCardView = !isCardView"
                class="inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50 cursor-pointer">
                <template v-if="!isCardView">
                  <TableCellsMerge size="16" />
                </template>
                <template v-else>
                  <AppWindowMac size="16" />
                </template>
              </Button>
            </div>
          </div>
        </div>
      </div>

      <!-- Product Count -->
      <div class="text-sm text-slate-500">
        Showing {{ visibleProducts.length }} of {{ filteredProducts.length }} products
      </div>

      <div v-if="isCardView">
        <!-- Product Grid -->
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
          <div v-for="product in visibleProducts" :key="product.id"
            class="overflow-hidden border border-slate-200 hover:shadow-md transition-shadow rounded-lg bg-white">
            <div :class="['p-6 space-y-4', product.visible ? '' : 'opacity-25']">
              <div class="flex justify-between items-start">
                <div v-if="product.editing" class="w-full">
                  <input v-model="tempEditValues[product.id].name"
                    class="w-full px-3 py-2 border border-slate-200 rounded-md font-semibold" />
                </div>
                <h3 v-else class="font-semibold text-lg line-clamp-2">{{ product.name }}</h3>
                <button @click="toggleVisibility(product.id)" class="text-slate-400 hover:text-slate-600 p-1"
                  :title="product.visible ? 'Hide Product' : 'Show Product'">
                  <component :is="product.visible ? Eye : EyeOff" :size="18" />
                </button>
              </div>

              <div class="space-y-2">
                <div class="flex justify-between items-center">
                  <span class="text-sm text-slate-500">Price</span>
                  <div v-if="product.editing">
                    <input v-model.number="tempEditValues[product.id].price" type="number" min="0" step="0.01"
                      class="w-24 px-3 py-1 border border-slate-200 rounded-md text-right" />
                  </div>
                  <span v-else class="font-medium flex flex-row items-center justify-center">
                    <IndianRupee :size="14" />{{ product.price.toFixed(2) }}
                  </span>
                </div>

                <div class="flex justify-between items-center">
                  <span class="text-sm text-slate-500">Stock</span>
                  <div class="flex items-center gap-2">
                    <div v-if="product.editing">
                      <input v-model.number="tempEditValues[product.id].stock" type="number" min="0"
                        class="w-20 px-3 py-1 border border-slate-200 rounded-md text-right" />
                    </div>
                    <span v-else class="font-medium">{{ product.stock }}</span>
                    <span class="px-2 py-1 text-xs font-medium rounded-full text-white" :class="getStockStatusClass(
                      product.editing ? tempEditValues[product.id].stock : product.stock,
                    )
                      ">
                      {{
                        getStockStatusLabel(
                          product.editing ? tempEditValues[product.id].stock : product.stock,
                        )
                      }}
                    </span>
                  </div>
                </div>

                <div class="flex justify-between items-center">
                  <span class="text-sm text-slate-500">Category</span>
                  <div v-if="product.editing">
                    <select v-model="tempEditValues[product.id].category"
                      class="w-32 px-3 py-1 border border-slate-200 rounded-md">
                      <option v-for="category in allCategories" :key="category" :value="category">
                        {{ category }}
                      </option>
                    </select>
                  </div>
                  <span v-else class="px-2 py-1 text-xs font-medium rounded-full border border-slate-200">
                    {{ product.category }}
                  </span>
                </div>
              </div>

              <div v-if="product.editing" class="flex gap-2">
                <button @click="saveInlineEdit(product.id)"
                  class="flex-1 inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-slate-800 text-white hover:bg-slate-700">
                  <Save :size="16" />
                  Update
                </button>
                <button @click="cancelInlineEdit(product.id)"
                  class="flex-1 inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
                  <X :size="16" />
                  Cancel
                </button>
              </div>
              <button v-else @click="startInlineEdit(product)"
                class="w-full inline-flex items-center justify-center gap-2 px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
                <Edit2 :size="16" />
                Edit Product
              </button>
            </div>
          </div>
        </div>
      </div>
      <div v-else>
        <!-- Product Table -->
        <div class="overflow-x-auto">
          <table class="w-full border-collapse border border-slate-200 bg-white rounded-lg min-w-[600px]">
            <thead class="bg-slate-100">
              <tr>
                <th class="border border-slate-200 p-3 text-left">Product</th>
                <th class="border border-slate-200 p-3 text-left">Price</th>
                <th class="border border-slate-200 p-3 text-left">Stock</th>
                <th class="border border-slate-200 p-3 text-left">Category</th>
                <th class="border border-slate-200 p-3 text-center">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="product in visibleProducts" :key="product.id"
                :class="['border-b border-slate-200', product.visible ? '' : 'opacity-25']">
                <td class="p-3">
                  <div v-if="product.editing">
                    <input v-model="tempEditValues[product.id].name"
                      class="w-full px-2 py-1 border border-slate-200 rounded-md font-semibold" />
                  </div>
                  <span v-else class="font-medium">{{ product.name }}</span>
                </td>
                <td class="p-3 text-right">
                  <div v-if="product.editing">
                    <input v-model.number="tempEditValues[product.id].price" type="number" min="0" step="0.01"
                      class="w-24 px-2 py-1 border border-slate-200 rounded-md text-right" />
                  </div>
                  <span v-else class="font-medium flex items-center">
                    <IndianRupee :size="14" />{{ product.price.toFixed(2) }}
                  </span>
                </td>
                <td class="p-3 flex justify-between flex-row items-center">
                  <span class="px-2 py-1 max-sm:px-0.5 text-xs font-medium rounded-full text-white" :class="getStockStatusClass(
                    product.editing ? tempEditValues[product.id].stock : product.stock,
                  )
                    ">
                    {{
                      getStockStatusLabel(
                        product.editing ? tempEditValues[product.id].stock : product.stock,
                      )
                    }}
                  </span>
                  <div v-if="product.editing">
                    <input v-model.number="tempEditValues[product.id].stock" type="number" min="0"
                      class="w-20 px-2 py-1 border border-slate-200 rounded-md text-right" />
                  </div>

                  <span v-else class="font-medium">{{ product.stock }}</span>
                </td>
                <td class="p-3">
                  <div v-if="product.editing">
                    <select v-model="tempEditValues[product.id].category"
                      class="w-full px-2 py-1 border border-slate-200 rounded-md">
                      <option v-for="category in allCategories" :key="category" :value="category">
                        {{ category }}
                      </option>
                    </select>
                  </div>
                  <span v-else class="px-2 py-1 text-xs font-medium rounded-full border border-slate-200">
                    {{ product.category }}
                  </span>
                </td>
                <td class="p-3 text-center flex gap-2 justify-center">
                  <button @click="toggleVisibility(product.id)" class="text-slate-400 hover:text-slate-600">
                    <component :is="product.visible ? Eye : EyeOff" :size="16" />
                  </button>
                  <button v-if="product.editing" @click="saveInlineEdit(product.id)"
                    class="px-3 py-1 text-sm font-medium bg-slate-800 text-white rounded-md hover:bg-slate-700 flex flex-row gap-2 justify-center items-center">
                    <Save :size="14" /> Save
                  </button>
                  <button v-if="product.editing" @click="cancelInlineEdit(product.id)"
                    class="px-3 py-1 text-sm font-medium bg-white border border-slate-200 rounded-md hover:bg-slate-50 flex flex-row gap-2 justify-center items-center">
                    <X :size="14" /> Cancel
                  </button>
                  <button v-else @click="startInlineEdit(product)"
                    class="px-3 py-1 text-sm font-medium bg-white border border-slate-200 rounded-md hover:bg-slate-50 flex flex-row gap-2 justify-center items-center">
                    <Edit2 :size="14" /> Edit
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>

      <!-- Load More / Infinite Scroll Trigger -->
      <div v-if="visibleProducts.length < filteredProducts.length" ref="loadMoreTrigger" class="py-8 text-center">
        <div v-if="isLoading" class="flex justify-center items-center gap-2">
          <Loader />
          <span>Loading more products...</span>
        </div>
      </div>

      <!-- Empty State -->
      <div v-if="filteredProducts.length === 0" class="text-center py-12">
        <div class="mx-auto w-24 h-24 bg-slate-100 rounded-full flex items-center justify-center mb-4">
          <Filter :size="32" class="text-slate-400" />
        </div>
        <h3 class="text-lg font-medium">No products found</h3>
        <p class="text-slate-500 mt-1">Try adjusting your filters or search term</p>
        <button @click="resetFilters"
          class="mt-4 inline-flex items-center justify-center px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
          Reset Filters
        </button>
      </div>
    </div>

    <!-- Filter Sheet -->
    <div v-if="isFilterSheetOpen" class="fixed inset-0 bg-[#00000082] bg-opacity-50 z-50 flex justify-start"
      @click.self="isFilterSheetOpen = false">
      <div class="bg-white w-full max-w-md h-full overflow-y-auto p-6 animate-slide-in-left">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-xl font-bold">Filter Products</h2>
          <button @click="isFilterSheetOpen = false" class="text-slate-400 hover:text-slate-600">
            <X :size="24" />
          </button>
        </div>

        <!-- Price Range Filter -->
        <div class="mb-6">
          <h3 class="text-sm font-medium mb-2">Price Range</h3>
          <div class="flex items-center justify-between mb-2">
            <span class="text-sm text-slate-500 flex flex-row justify-center items-center">
              <IndianRupee :size="16" />{{ priceRange[0].toFixed(2) }}
            </span>
            <span class="text-sm text-slate-500 flex flex-row justify-center items-center">
              <IndianRupee :size="16" />{{ priceRange[1].toFixed(2) }}
            </span>
          </div>
          <Slider v-model="priceRange" :max="maxPrice" :step="1" />
        </div>

        <!-- Stock Range Filter -->
        <div class="mb-6">
          <h3 class="text-sm font-medium mb-2">Stock Range</h3>
          <div class="flex items-center justify-between mb-2">
            <span class="text-sm text-slate-500">{{ stockRange[0] }}</span>
            <span class="text-sm text-slate-500">{{ stockRange[1] }}</span>
          </div>
          <!-- <input type="range" v-model.number="stockRange[0]" :min="0" :max="maxStock" class="w-full mb-2" />
          <input type="range" v-model.number="stockRange[1]" :min="0" :max="maxStock" class="w-full" /> -->
          <Slider v-model="stockRange" :max="maxStock" :step="1" />
        </div>

        <!-- Category Filter -->
        <div class="mb-6">
          <h3 class="text-sm font-medium mb-2">Categories</h3>
          <div class="grid grid-cols-2 gap-2">
            <div v-for="category in allCategories" :key="category" class="flex items-center space-x-2">
              <input type="checkbox" :id="`category-${category}`" :value="category" v-model="selectedCategories"
                class="rounded border-slate-300 text-slate-800 focus:ring-slate-500" />
              <label :for="`category-${category}`">{{ category }}</label>
            </div>
          </div>
        </div>

        <!-- Derived Filters -->
        <div class="mb-6">
          <h3 class="text-sm font-medium mb-2">Special Filters</h3>
          <div class="space-y-2">
            <div class="flex items-center space-x-2">
              <input type="checkbox" id="low-stock" v-model="showLowStockOnly"
                class="rounded border-slate-300 text-slate-800 focus:ring-slate-500" />
              <label for="low-stock">Only Show Low Stock</label>
            </div>
            <div class="flex items-center space-x-2">
              <input type="checkbox" id="out-of-stock" v-model="showOutOfStockOnly"
                class="rounded border-slate-300 text-slate-800 focus:ring-slate-500" />
              <label for="out-of-stock">Only Show Out of Stock</label>
            </div>
          </div>
        </div>

        <!-- Sort Options -->
        <div class="mb-6">
          <h3 class="text-sm font-medium mb-2">Sort By</h3>
          <div class="flex items-center gap-2">
            <select v-model="sortBy" class="w-full px-3 py-2 border border-slate-200 rounded-md">
              <option value="name">Name</option>
              <option value="price">Price</option>
              <option value="stock">Stock</option>
              <option value="category">Category</option>
            </select>
            <button @click="toggleSortDirection" class="p-2 border border-slate-200 rounded-md">
              <component :is="sortDirection === 'asc' ? ChevronUp : ChevronDown" :size="16" />
            </button>
          </div>
        </div>

        <button @click="resetFilters"
          class="w-full inline-flex items-center justify-center px-4 py-2 rounded-md text-sm font-medium bg-white border border-slate-200 hover:bg-slate-50">
          Reset Filters
        </button>
      </div>
    </div>

    <!-- Toast Notifications -->
    <div class="fixed bottom-4 right-4 z-50">
      <div v-for="(toast, index) in toasts" :key="index" class="mb-2 p-4 rounded-lg shadow-lg max-w-md animate-fade-in"
        :class="toast.type === 'success'
          ? 'bg-green-50 border-l-4 border-green-500 text-green-800'
          : 'bg-red-50 border-l-4 border-red-500 text-red-800'
          ">
        <div class="flex items-start">
          <div class="flex-shrink-0">
            <component :is="toast.type === 'success' ? CheckCircle : AlertCircle" :size="20" class="mt-0.5" />
          </div>
          <div class="ml-3">
            <p class="text-sm font-medium">{{ toast.title }}</p>
            <p class="text-sm mt-1">{{ toast.message }}</p>
          </div>
          <button @click="removeToast(index)" class="ml-auto text-slate-400 hover:text-slate-600">
            <X :size="16" />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick, watch } from 'vue'
import { Slider } from './ui/slider'
import {
  Search,
  Filter,
  Edit2,
  Save,
  X,
  ChevronDown,
  ChevronUp,
  Eye,
  EyeOff,
  SlidersHorizontal,
  Upload,
  CheckCircle,
  AlertCircle,
  IndianRupee,
  TableCellsMerge,
  AppWindowMac,
} from 'lucide-vue-next'
import Loader from '../common/Loader.vue'
import { initialProducts } from '../../data/products'

// State
const products = ref(initialProducts.map((p) => ({ ...p, editing: false, visible: true })))
const searchTerm = ref('')
const tempEditValues = ref({})
const isFilterSheetOpen = ref(false)
const fileInput = ref(null)
const loadMoreTrigger = ref(null)
const isLoading = ref(false)
const toasts = ref([])
const visibleCount = ref(12) // Initial number of visible products
let observer = null
const isCardView = ref(true)

console.log(products.value)

watch(
  products,
  (newVal, oldVal) => {
    console.log('Products changed')
    console.log(newVal)
    console.log(oldVal)
  },
  { deep: true },
)

// Filter states
const priceRange = ref([0, 10000])
const stockRange = ref([0, 50])
const selectedCategories = ref([])
const showLowStockOnly = ref(false)
const showOutOfStockOnly = ref(false)
const sortBy = ref('name')
const sortDirection = ref('asc')
const showHiddenProduct = ref(false)

// Computed properties
const allCategories = computed(() => {
  return [...new Set(products.value.map((product) => product.category))]
})

const maxPrice = computed(() => {
  return Math.max(...products.value.map((p) => p.price))
})

const maxStock = computed(() => {
  return Math.max(...products.value.map((p) => p.stock))
})

const sortByLabel = computed(() => {
  return sortBy.value.charAt(0).toUpperCase() + sortBy.value.slice(1)
})

const filteredProducts = computed(() => {
  let result = [...products.value]
  if (searchTerm.value) {
    result = result.filter(
      (product) =>
        product.name.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
        product.category.toLowerCase().includes(searchTerm.value.toLowerCase()),
    )
  }

  result = result.filter(
    (product) => product.price >= priceRange.value[0] && product.price <= priceRange.value[1],
  )
  result = result.filter(
    (product) => product.stock >= stockRange.value[0] && product.stock <= stockRange.value[1],
  )

  if (selectedCategories.value.length > 0) {
    result = result.filter((product) => selectedCategories.value.includes(product.category))
  }

  if (showLowStockOnly.value) {
    result = result.filter((product) => product.stock > 0 && product.stock <= 10)
  }
  if (showOutOfStockOnly.value) {
    result = result.filter((product) => product.stock === 0)
  }

  // Visibility filter
  // result = result.filter((product) => product.visible)
  result.sort((a, b) => {
    let comparison = 0

    if (sortBy.value === 'name') {
      comparison = a.name.localeCompare(b.name)
    } else if (sortBy.value === 'price') {
      comparison = a.price - b.price
    } else if (sortBy.value === 'stock') {
      comparison = a.stock - b.stock
    } else if (sortBy.value === 'category') {
      comparison = a.category.localeCompare(b.category)
    }

    return sortDirection.value === 'asc' ? comparison : -comparison
  })

  return result
})

const visibleProducts = computed(() => {
  return filteredProducts.value.slice(0, visibleCount.value)
})

// Methods
const toggleVisibility = (id) => {
  products.value = products.value.map((product) =>
    product.id === id ? { ...product, visible: !product.visible } : product,
  )
}

const startInlineEdit = (product) => {
  tempEditValues.value = {
    ...tempEditValues.value,
    [product.id]: { ...product },
  }

  products.value = products.value.map((p) => (p.id === product.id ? { ...p, editing: true } : p))
}

const cancelInlineEdit = (id) => {
  products.value = products.value.map((p) => (p.id === id ? { ...p, editing: false } : p))

  // Remove temporary values
  const newTempValues = { ...tempEditValues.value }
  delete newTempValues[id]
  tempEditValues.value = newTempValues
}

const saveInlineEdit = (id) => {
  const editedProduct = tempEditValues.value[id]

  // Validate
  if (editedProduct.price < 0) {
    showToast('Validation Error', 'Price must be greater than or equal to 0', 'error')
    return
  }

  if (editedProduct.stock < 0) {
    showToast('Validation Error', 'Stock must be greater than or equal to 0', 'error')
    return
  }

  // Update product
  products.value = products.value.map((p) =>
    p.id === id ? { ...editedProduct, editing: false } : p,
  )

  // Remove temporary values
  const newTempValues = { ...tempEditValues.value }
  delete newTempValues[id]
  tempEditValues.value = newTempValues

  showToast('Product Updated', `${editedProduct.name} has been updated successfully.`, 'success')
}

const resetFilters = () => {
  searchTerm.value = ''
  priceRange.value = [0, maxPrice.value]
  stockRange.value = [0, maxStock.value]
  selectedCategories.value = []
  showLowStockOnly.value = false
  showOutOfStockOnly.value = false
  sortBy.value = 'name'
  sortDirection.value = 'asc'
  isFilterSheetOpen.value = false
}

const getStockStatusLabel = (stock) => {
  if (stock === 0) return 'Out of Stock'
  if (stock <= 10) return 'Low Stock'
  if (stock <= 20) return 'Medium Stock'
  return 'High Stock'
}

const getStockStatusClass = (stock) => {
  if (stock === 0) return 'bg-red-500'
  if (stock <= 10) return 'bg-green-500'
  if (stock <= 20) return 'bg-yellow-500'
  return 'bg-red-500'
}

const toggleSortDirection = () => {
  sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
}

const handleFileUpload = (event) => {
  const file = event.target.files?.[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const content = e.target?.result
      const lines = content.split('\n')
      const headers = lines[0].split(',').map((h) => h.trim())

      const nameIndex = headers.indexOf('name')
      const priceIndex = headers.indexOf('price')
      const stockIndex = headers.indexOf('stock')
      const categoryIndex = headers.indexOf('category')

      if (nameIndex === -1 || priceIndex === -1 || stockIndex === -1 || categoryIndex === -1) {
        showToast(
          'Invalid CSV Format',
          'The CSV file must contain name, price, stock, and category columns.',
          'error',
        )
        return
      }

      const newProducts = []

      for (let i = 1; i < lines.length; i++) {
        if (!lines[i].trim()) continue

        const values = lines[i].split(',').map((v) => v.trim())

        const product = {
          id: products.value.length + newProducts.length + 1,
          name: values[nameIndex],
          price: parseFloat(values[priceIndex]) || 0,
          stock: parseInt(values[stockIndex]) || 0,
          category: values[categoryIndex],
          visible: true,
          editing: false,
        }

        newProducts.push(product)
      }

      if (newProducts.length > 0) {
        products.value = [...products.value, ...newProducts]
        filteredProducts.value = [...products.value]
        nextTick(() => {
          reattachObserver()
        })
        showToast(
          'Products Imported',
          `Successfully imported ${newProducts.length} products.`,
          'success',
        )
      } else {
        showToast('No Products Found', 'No valid products were found in the CSV file.', 'error')
      }
    } catch (error) {
      showToast('Error Parsing File', 'There was an error parsing the uploaded file.', 'error')
    }

    // Reset file input
    if (fileInput.value) {
      fileInput.value.value = ''
    }
  }

  reader.readAsText(file)
}

const loadMoreProducts = () => {
  if (isLoading.value || visibleCount.value >= filteredProducts.value.length) return

  isLoading.value = true

  // Simulate loading delay
  setTimeout(() => {
    visibleCount.value += 10
    isLoading.value = false
  }, 500)
}

const showToast = (title, message, type = 'success') => {
  const toast = { title, message, type }
  toasts.value.push(toast)

  // Auto-remove toast after 5 seconds
  setTimeout(() => {
    removeToast(toasts.value.indexOf(toast))
  }, 5000)
}

const removeToast = (index) => {
  if (index > -1) {
    toasts.value.splice(index, 1)
  }
}

watch(filteredProducts, () => {
  visibleCount.value = 10

  reattachObserver()
})

// Intersection Observer: Infinite Scroll
const reattachObserver = () => {
  if (observer) observer.disconnect()
  observer = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting && !isLoading.value) {
        loadMoreProducts()
      }
    },
    { threshold: 0.1 },
  )

  nextTick(() => {
    if (loadMoreTrigger.value) observer.observe(loadMoreTrigger.value) // 🔥 Ensure new elements are watched
  })
}

onMounted(() => {
  reattachObserver()
})
</script>

<style scoped>
.animate-slide-in-left {
  animation: slideInLeft 0.4s cubic-bezier(0.25, 0.1, 0.25, 1) both;
}

.animate-fade-in {
  animation: fadeIn 0.4s ease-out both;
}

@keyframes slideInLeft {
  from {
    transform: translateX(-30px);
    opacity: 0;
  }

  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}
</style>