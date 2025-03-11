<script setup>
import { ref, onMounted } from 'vue';
import ProductsSection from './components/ProductsSection.vue';
import CustomersSection from './components/CustomersSection.vue';
import SalesSection from './components/SalesSection.vue';

// API base URL - change this to your actual API endpoint
const API_BASE_URL = 'http://localhost:25565/api';

// Page state
const currentPage = ref('products');

// Data state
const products = ref([]);
const customers = ref([]);

// Fetch data on component mount
onMounted(() => {
  fetchProducts();
  fetchCustomers();
});

// API Functions for Products
async function fetchProducts() {
  try {
    const response = await fetch(`${API_BASE_URL}/product/`);
    if (!response.ok) throw new Error('Failed to fetch products');
    products.value = await response.json();
  } catch (error) {
    console.error('Error fetching products:', error);
  }
}

// API Functions for Customers
async function fetchCustomers() {
  try {
    const response = await fetch(`${API_BASE_URL}/customer/`);
    if (!response.ok) throw new Error('Failed to fetch customers');
    customers.value = await response.json();
  } catch (error) {
    console.error('Error fetching customers:', error);
  }
}

// Refresh data when changes occur
function handleDataRefresh() {
  fetchProducts();
  fetchCustomers();
}
</script>

<template>
  <div class="min-h-screen bg-gray-50">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between h-16">
          <div class="flex">
            <div class="flex-shrink-0 flex items-center">
              <h1 class="text-xl font-bold text-gray-900">Sheaz Grace Inventory Manager</h1>
            </div>
            <nav class="ml-6 flex space-x-8">
              <button
                  @click="currentPage = 'products'"
                  :class="[
                  'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium',
                  currentPage === 'products' 
                    ? 'border-indigo-500 text-gray-900' 
                    : 'border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700'
                ]"
              >
                Products
              </button>
              <button
                  @click="currentPage = 'customers'"
                  :class="[
                  'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium',
                  currentPage === 'customers' 
                    ? 'border-indigo-500 text-gray-900' 
                    : 'border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700'
                ]"
              >
                Customers
              </button>
              <button
                  @click="currentPage = 'sales'"
                  :class="[
                  'inline-flex items-center px-1 pt-1 border-b-2 text-sm font-medium',
                  currentPage === 'sales'
                    ? 'border-indigo-500 text-gray-900' 
                    : 'border-transparent text-gray-500 hover:border-gray-300 hover:text-gray-700'
                ]"
              >
                Sales
              </button>
            </nav>
          </div>
        </div>
      </div>
    </header>

    <main class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
      <!-- Products Page -->
      <ProductsSection
          v-if="currentPage === 'products'"
          :API_BASE_URL="API_BASE_URL"
          @refresh-data="handleDataRefresh"
      />

      <!-- Customers Page -->
      <CustomersSection
          v-if="currentPage === 'customers'"
          :API_BASE_URL="API_BASE_URL"
          @refresh-data="handleDataRefresh"
      />

      <!-- Sales Page -->
      <SalesSection
          v-if="currentPage === 'sales'"
          :API_BASE_URL="API_BASE_URL"
          :customers="customers"
          :products="products"
          @refresh-data="handleDataRefresh"
      />
    </main>
  </div>
</template>
