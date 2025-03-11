<script setup>
import { ref, reactive } from 'vue';

const props = defineProps({
  API_BASE_URL: {
    type: String,
    required: true
  }
});

const emit = defineEmits(['refresh-data']);

// Local state
const showProductForm = ref(false);
const isLoading = ref(false);
const errors = ref(null);
const products = ref([]);

// Form state
const newProduct = reactive({
  name: '',
  price: 0,
  quantity: 0
});

// API Functions
async function fetchProducts() {
  isLoading.value = true;
  errors.value = null;

  try {
    const response = await fetch(`${props.API_BASE_URL}/product/`);
    if (!response.ok) throw new Error('Failed to fetch products');

    const data = await response.json();
    products.value = data;
  } catch (error) {
    console.error('Error fetching products:', error);
    errors.value = error.message;
  } finally {
    isLoading.value = false;
  }
}

async function addProduct() {
  if (!newProduct.name || newProduct.price <= 0 || newProduct.quantity <= 0) {
    alert('Please fill in all product fields with valid values');
    return;
  }

  isLoading.value = true;

  try {
    const response = await fetch(`${props.API_BASE_URL}/product/`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: newProduct.name,
        price: parseFloat(newProduct.price),
        quantity: parseInt(newProduct.quantity)
      })
    });

    if (!response.ok) throw new Error('Failed to add product');

    // Refresh products list
    await fetchProducts();
    emit('refresh-data');

    // Reset form
    newProduct.name = '';
    newProduct.price = 0;
    newProduct.quantity = 0;
    showProductForm.value = false;
  } catch (error) {
    console.error('Error adding product:', error);
    alert(`Error adding product: ${error.message}`);
  } finally {
    isLoading.value = false;
  }
}

async function removeProduct(productId) {
  if (!confirm('Are you sure you want to delete this product?')) return;

  isLoading.value = true;

  try {
    const response = await fetch(`${props.API_BASE_URL}/product/${productId}`, {
      method: 'DELETE'
    });

    if (!response.ok) throw new Error('Failed to delete product');

    // Refresh products list
    await fetchProducts();
    emit('refresh-data');
  } catch (error) {
    console.error('Error deleting product:', error);
    alert(`Error deleting product: ${error.message}`);
  } finally {
    isLoading.value = false;
  }
}

// Fetch data on component mount
fetchProducts();
</script>

<template>
  <div>
    <div class="px-4 py-5 sm:px-6 flex justify-between items-center">
      <div>
        <h2 class="text-lg font-medium text-gray-900">Products</h2>
        <p class="mt-1 text-sm text-gray-500">Manage your product inventory</p>
      </div>
      <button
          @click="showProductForm = true"
          class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Add Product
      </button>
    </div>

    <!-- Loading and Error States -->
    <div v-if="isLoading" class="flex justify-center py-8">
      <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
    </div>

    <div v-else-if="errors" class="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded mx-4 mb-6">
      <p>{{ errors }}</p>
      <button
          @click="fetchProducts"
          class="mt-2 text-sm text-red-700 underline"
      >
        Try again
      </button>
    </div>

    <!-- Product Form -->
    <div v-if="showProductForm" class="bg-white shadow sm:rounded-lg mb-6 mx-4">
      <div class="px-4 py-5 sm:p-6">
        <h3 class="text-lg font-medium leading-6 text-gray-900">Add New Product</h3>
        <div class="mt-5 grid grid-cols-1 gap-4 sm:grid-cols-2">
          <div>
            <label for="productName" class="block text-sm font-medium text-gray-700">Name</label>
            <input
                type="text"
                id="productName"
                v-model="newProduct.name"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            />
          </div>
          <div>
            <label for="productPrice" class="block text-sm font-medium text-gray-700">Price</label>
            <input
                type="number"
                id="productPrice"
                v-model="newProduct.price"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            />
          </div>
          <div>
            <label for="productQuantity" class="block text-sm font-medium text-gray-700">Quantity</label>
            <input
                type="number"
                id="productQuantity"
                v-model="newProduct.quantity"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            />
          </div>
        </div>
        <div class="mt-5 flex justify-end">
          <button
              @click="showProductForm = false"
              class="mr-3 inline-flex items-center px-4 py-2 border border-gray-300 shadow-sm text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            Cancel
          </button>
          <button
              @click="addProduct"
              :disabled="isLoading"
              class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50"
          >
            <span v-if="isLoading">Saving...</span>
            <span v-else>Save</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Products Table -->
    <div v-if="!isLoading && !errors" class="bg-white shadow overflow-hidden sm:rounded-lg mx-4">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
        <tr>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Product ID
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Name
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Price
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Quantity
          </th>
          <th scope="col" class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">
            Actions
          </th>
        </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
        <tr v-for="product in products" :key="product.id">
          <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
            {{ product.id }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ product.name }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            ${{ product.price.toFixed(2) }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ product.quantity }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
            <button
                @click="removeProduct(product.id)"
                :disabled="isLoading"
                class="text-red-600 hover:text-red-900 disabled:opacity-50"
            >
              Remove
            </button>
          </td>
        </tr>
        <tr v-if="products.length === 0">
          <td colspan="5" class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">
            No products found. Add a product to get started.
          </td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template> 