<script setup>
import { ref, reactive } from 'vue';
import InvoiceModal from './InvoiceModal.vue';

const props = defineProps({
  API_BASE_URL: {
    type: String,
    required: true
  },
  customers: {
    type: Array,
    required: true
  },
  products: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['refresh-data']);

// Local state
const showSaleForm = ref(false);
const showInvoice = ref(false);
const currentInvoice = ref({});
const isLoading = ref(false);
const errors = ref(null);
const sales = ref([]);

// Form state
const newSale = reactive({
  customer: {id: "", email: "", name: ""},
  product: {id:"", price: 0, quantity: 0, name: ""},
  quantity: 1,
  total: 0
});

// API Functions
async function fetchSales() {
  isLoading.value = true;
  errors.value = null;

  try {
    const response = await fetch(`${props.API_BASE_URL}/sale/`);
    if (!response.ok) throw new Error('Failed to fetch sales');

    const data = await response.json();
    sales.value = data;
  } catch (error) {
    console.error('Error fetching sales:', error);
    errors.value = error.message;
  } finally {
    isLoading.value = false;
  }
}

function updateSalePrice() {
  if (!newSale.product || !newSale.quantity) {
    newSale.total = 0;
    return;
  }

  const product = props.products.find(p => p.id === newSale.product.id);
  if (product) {
    newSale.total = product.price * newSale.quantity;
  }
}

async function createSale() {
  if (!newSale.customer || !newSale.product || !newSale.quantity || newSale.quantity <= 0) {
    alert('Please fill in all fields with valid values');
    return;
  }

  const product = props.products.find(p => p.id === newSale.product.id);
  console.log(product);
  if (!product) {
    alert('Selected product not found');
    return;
  }

  const customer = props.customers.find(p => p.id === newSale.customer.id);
  console.log(customer);
  if (!customer) {
    alert('Selected customer not found');
    return;
  }

  if (newSale.quantity > product.quantity) {
    alert(`Not enough inventory. Only ${product.quantity} units available.`);
    return;
  }

  isLoading.value = true;

  try {
    // Create sale
    const saleData = {
      date: new Date().toISOString(),
      customer: customer,
      product: product,
      quantity: parseInt(newSale.quantity),
      total: newSale.total
    };

    const response = await fetch(`${props.API_BASE_URL}/sale/`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(saleData)
    });

    if (!response.ok) throw new Error('Failed to create sale');

    const createdSale = await response.json();

    // Update product inventory
    const updatedProduct = {
      ...product,
      quantity: product.quantity - saleData.quantity
    };

    const productResponse = await fetch(`${props.API_BASE_URL}/product/${product.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(updatedProduct)
    });

    if (!productResponse.ok) throw new Error('Failed to update product inventory');

    // Refresh data
    await fetchSales();
    emit('refresh-data');

    // Reset form
    newSale.customer= {id: "", email: "", name: ""};
    newSale.product= {id:"", price: 0, quantity: 0, name: ""};
    newSale.quantity = 1;
    newSale.total = 0;
    showSaleForm.value = false;

    // Show invoice
    viewInvoice(createdSale);
  } catch (error) {
    console.error('Error creating sale:', error);
    alert(`Error creating sale: ${error.message}`);
  } finally {
    isLoading.value = false;
  }
}

function viewInvoice(sale) {
  currentInvoice.value = sale;
  showInvoice.value = true;
}

function formatDate(date) {
  if (!date) return '';
  const d = new Date(date);
  return d.toLocaleDateString();
}

function getCustomerName(customerId) {
  const customer = props.customers.find(c => c.id === customerId);
  return customer ? customer.name : 'Unknown Customer';
}

function getCustomerEmail(customerId) {
  const customer = props.customers.find(c => c.id === customerId);
  return customer ? customer.email : '';
}

function getProductName(productId) {
  const product = props.products.find(p => p.id === productId);
  return product ? product.name : 'Unknown Product';
}

function getProductPrice(productId) {
  const product = props.products.find(p => p.id === productId);
  return product ? product.price : 0;
}

// Fetch data on component mount
fetchSales();
</script>

<template>
  <div>
    <div class="px-4 py-5 sm:px-6 flex justify-between items-center">
      <div>
        <h2 class="text-lg font-medium text-gray-900">Sales</h2>
        <p class="mt-1 text-sm text-gray-500">Create sales and generate invoices</p>
      </div>
      <button
          @click="showSaleForm = true"
          class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        New Sale
      </button>
    </div>

    <!-- Loading and Error States -->
    <div v-if="isLoading" class="flex justify-center py-8">
      <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
    </div>

    <div v-else-if="errors" class="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded mx-4 mb-6">
      <p>{{ errors }}</p>
      <button
          @click="fetchSales"
          class="mt-2 text-sm text-red-700 underline"
      >
        Try again
      </button>
    </div>

    <!-- Sale Form -->
    <div v-if="showSaleForm" class="bg-white shadow sm:rounded-lg mb-6 mx-4">
      <div class="px-4 py-5 sm:p-6">
        <h3 class="text-lg font-medium leading-6 text-gray-900">Create New Sale</h3>
        <div class="mt-5 grid grid-cols-1 gap-4 sm:grid-cols-2">
          <div>
            <label for="customerSelect" class="block text-sm font-medium text-gray-700">Customer</label>
            <select
                id="customerSelect"
                v-model="newSale.customer"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            >
              <option value="" disabled>Select a customer</option>
              <option v-for="customer in customers" :key="customer.id" :value="customer">
                {{ customer.name }} ({{ customer.email }})
              </option>
            </select>
          </div>
          <div>
            <label for="productSelect" class="block text-sm font-medium text-gray-700">Product</label>
            <select
                id="productSelect"
                v-model="newSale.product"
                @change="updateSalePrice"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            >
              <option value="" disabled>Select a product</option>
              <option v-for="product in products" :key="product.id" :value="product">
                {{ product.name }} - ${{ product.price.toFixed(2) }} ({{ product.quantity }} in stock)
              </option>
            </select>
          </div>
          <div>
            <label for="saleQuantity" class="block text-sm font-medium text-gray-700">Quantity</label>
            <input
                type="number"
                id="saleQuantity"
                v-model="newSale.quantity"
                @input="updateSalePrice"
                min="1"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            />
          </div>
          <div>
            <label class="block text-sm font-medium text-gray-700">Total Price</label>
            <div class="mt-1 block w-full py-2 px-3 sm:text-sm">
              ${{ newSale.total.toFixed(2) }}
            </div>
          </div>
        </div>
        <div class="mt-5 flex justify-end">
          <button
              @click="showSaleForm = false"
              class="mr-3 inline-flex items-center px-4 py-2 border border-gray-300 shadow-sm text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            Cancel
          </button>
          <button
              @click="createSale"
              :disabled="isLoading"
              class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50"
          >
            <span v-if="isLoading">Processing...</span>
            <span v-else>Complete Sale</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Sales Table -->
    <div v-if="!isLoading && !errors" class="bg-white shadow overflow-hidden sm:rounded-lg mx-4">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
        <tr>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Invoice #
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Date
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Customer
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Product
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Total
          </th>
          <th scope="col" class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">
            Actions
          </th>
        </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
        <tr v-for="sale in sales" :key="sale.id">
          <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
            #{{ sale.id }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ formatDate(sale.date) }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ getCustomerName(sale.customer.id) }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ getProductName(sale.product.id) }} ({{ sale.quantity }})
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            ${{ sale.total.toFixed(2) }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
            <button
                @click="viewInvoice(sale); console.log(sale);"
                class="text-indigo-600 hover:text-indigo-900 mr-4"
            >
              View Invoice
            </button>
          </td>
        </tr>
        <tr v-if="sales.length === 0">
          <td colspan="6" class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">
            No sales found. Create a sale to get started.
          </td>
        </tr>
        </tbody>
      </table>
    </div>

    <!-- Invoice Modal -->
    <InvoiceModal
        :show="showInvoice"
        :invoice="currentInvoice"
        :get-customer-name="getCustomerName"
        :get-customer-email="getCustomerEmail"
        :get-product-name="getProductName"
        :get-product-price="getProductPrice"
        :format-date="formatDate"
        @close="showInvoice = false"
    />
  </div>
</template> 