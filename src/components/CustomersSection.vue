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
const showCustomerForm = ref(false);
const isLoading = ref(false);
const errors = ref(null);
const customers = ref([]);

// Form state
const newCustomer = reactive({
  name: '',
  email: ''
});

// API Functions
async function fetchCustomers() {
  isLoading.value = true;
  errors.value = null;

  try {
    const response = await fetch(`${props.API_BASE_URL}/customer/`);
    if (!response.ok) throw new Error('Failed to fetch customers');

    const data = await response.json();
    customers.value = data;
  } catch (error) {
    console.error('Error fetching customers:', error);
    errors.value = error.message;
  } finally {
    isLoading.value = false;
  }
}

async function addCustomer() {
  if (!newCustomer.name || !newCustomer.email) {
    alert('Please fill in all customer fields');
    return;
  }

  // Simple email validation
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(newCustomer.email)) {
    alert('Please enter a valid email address');
    return;
  }

  isLoading.value = true;

  try {
    const response = await fetch(`${props.API_BASE_URL}/customer/`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        name: newCustomer.name,
        email: newCustomer.email
      })
    });

    if (!response.ok) throw new Error('Failed to add customer');

    // Refresh customers list
    await fetchCustomers();
    emit('refresh-data');

    // Reset form
    newCustomer.name = '';
    newCustomer.email = '';
    showCustomerForm.value = false;
  } catch (error) {
    console.error('Error adding customer:', error);
    alert(`Error adding customer: ${error.message}`);
  } finally {
    isLoading.value = false;
  }
}

async function removeCustomer(customerId) {
  if (!confirm('Are you sure you want to delete this customer?')) return;

  isLoading.value = true;

  try {
    const response = await fetch(`${props.API_BASE_URL}/customers/${customerId}`, {
      method: 'DELETE'
    });

    if (!response.ok) throw new Error('Failed to delete customer');

    // Refresh customers list
    await fetchCustomers();
    emit('refresh-data');
  } catch (error) {
    console.error('Error deleting customer:', error);
    alert(`Error deleting customer: ${error.message}`);
  } finally {
    isLoading.value = false;
  }
}

// Fetch data on component mount
fetchCustomers();
</script>

<template>
  <div>
    <div class="px-4 py-5 sm:px-6 flex justify-between items-center">
      <div>
        <h2 class="text-lg font-medium text-gray-900">Customers</h2>
        <p class="mt-1 text-sm text-gray-500">Manage your customer database</p>
      </div>
      <button
          @click="showCustomerForm = true"
          class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
      >
        Add Customer
      </button>
    </div>

    <!-- Loading and Error States -->
    <div v-if="isLoading" class="flex justify-center py-8">
      <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-indigo-600"></div>
    </div>

    <div v-else-if="errors" class="bg-red-50 border border-red-200 text-red-700 px-4 py-3 rounded mx-4 mb-6">
      <p>{{ errors }}</p>
      <button
          @click="fetchCustomers"
          class="mt-2 text-sm text-red-700 underline"
      >
        Try again
      </button>
    </div>

    <!-- Customer Form -->
    <div v-if="showCustomerForm" class="bg-white shadow sm:rounded-lg mb-6 mx-4">
      <div class="px-4 py-5 sm:p-6">
        <h3 class="text-lg font-medium leading-6 text-gray-900">Add New Customer</h3>
        <div class="mt-5 grid grid-cols-1 gap-4 sm:grid-cols-2">
          <div>
            <label for="customerName" class="block text-sm font-medium text-gray-700">Name</label>
            <input
                type="text"
                id="customerName"
                v-model="newCustomer.name"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            />
          </div>
          <div>
            <label for="customerEmail" class="block text-sm font-medium text-gray-700">Email</label>
            <input
                type="email"
                id="customerEmail"
                v-model="newCustomer.email"
                class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            />
          </div>
        </div>
        <div class="mt-5 flex justify-end">
          <button
              @click="showCustomerForm = false"
              class="mr-3 inline-flex items-center px-4 py-2 border border-gray-300 shadow-sm text-sm font-medium rounded-md text-gray-700 bg-white hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            Cancel
          </button>
          <button
              @click="addCustomer"
              :disabled="isLoading"
              class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50"
          >
            <span v-if="isLoading">Saving...</span>
            <span v-else>Save</span>
          </button>
        </div>
      </div>
    </div>

    <!-- Customers Table -->
    <div v-if="!isLoading && !errors" class="bg-white shadow overflow-hidden sm:rounded-lg mx-4">
      <table class="min-w-full divide-y divide-gray-200">
        <thead class="bg-gray-50">
        <tr>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Customer ID
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Name
          </th>
          <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
            Email
          </th>
          <th scope="col" class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">
            Actions
          </th>
        </tr>
        </thead>
        <tbody class="bg-white divide-y divide-gray-200">
        <tr v-for="customer in customers" :key="customer.id">
          <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">
            {{ customer.id }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ customer.name }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
            {{ customer.email }}
          </td>
          <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
            <button
                @click="removeCustomer(customer.id)"
                :disabled="isLoading"
                class="text-red-600 hover:text-red-900 disabled:opacity-50"
            >
              Remove
            </button>
          </td>
        </tr>
        <tr v-if="customers.length === 0">
          <td colspan="4" class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-center">
            No customers found. Add a customer to get started.
          </td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template> 