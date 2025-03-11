<script setup>
import { defineProps } from 'vue';

const props = defineProps({
  invoice: {
    type: Object,
    required: true
  },
  show: {
    type: Boolean,
    required: true
  },
  getCustomerName: {
    type: Function,
    required: true
  },
  getCustomerEmail: {
    type: Function,
    required: true
  },
  getProductName: {
    type: Function,
    required: true
  },
  getProductPrice: {
    type: Function,
    required: true
  },
  formatDate: {
    type: Function,
    required: true
  }
});

const emit = defineEmits(['close']);

function printInvoice() {
  window.print();
}
</script>

<template>
  <div v-if="show" class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-lg shadow-xl max-w-4xl w-full max-h-[90vh] overflow-y-auto">
      <div class="p-6">
        <div class="flex justify-between items-start">
          <h2 class="text-2xl font-bold text-gray-900">INVOICE #{{ invoice.id }}</h2>
          <button @click="$emit('close')" class="text-gray-400 hover:text-gray-500">
            <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>

        <div class="mt-6 grid grid-cols-2 gap-6">
          <div>
            <h3 class="text-gray-500 text-sm font-medium text-gray-700">Invoice To:</h3>
            <div class="mt-2">
              <p class="font-medium text-gray-700">{{ getCustomerName(invoice.customer.id) }}</p>
              <p class="text-gray-500">{{ getCustomerEmail(invoice.customer.id) }}</p>
            </div>
          </div>
          <div class="text-right">
            <h3 class="text-gray-500 text-sm font-medium text-gray-700">Invoice Details:</h3>
            <div class="mt-2">
              <p class="text-gray-500"><span class="font-medium text-gray-700">Date:</span> {{ formatDate(invoice.date) }}</p>
              <p class="text-gray-500"><span class="font-medium text-gray-700">Invoice #:</span> {{ invoice.id }}</p>
            </div>
          </div>
        </div>

        <div class="mt-8">
          <table class="min-w-full divide-y divide-gray-200">
            <thead>
            <tr>
              <th scope="col" class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-700 text-gray-500 uppercase tracking-wider">
                Item
              </th>
              <th scope="col" class="px-6 py-3 bg-gray-50 text-right text-xs font-medium text-gray-700 text-gray-500 uppercase tracking-wider">
                Quantity
              </th>
              <th scope="col" class="px-6 py-3 bg-gray-50 text-right text-xs font-medium text-gray-700 text-gray-500 uppercase tracking-wider">
                Unit Price
              </th>
              <th scope="col" class="px-6 py-3 bg-gray-50 text-right text-xs font-medium text-gray-700 text-gray-500 uppercase tracking-wider">
                Amount
              </th>
            </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-200">
            <tr>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-700 text-gray-900">
                {{ getProductName(invoice.product.id) }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-right">
                {{ invoice.quantity }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-right">
                ${{ getProductPrice(invoice.product.id).toFixed(2) }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500 text-right">
                ${{ (getProductPrice(invoice.product.id) * invoice.quantity).toFixed(2) }}
              </td>
            </tr>
            </tbody>
          </table>
        </div>

        <div class="mt-8 border-t border-gray-200 pt-6">
          <div class="flex justify-between text-sm">
            <span class="font-medium text-gray-700">Subtotal:</span>
            <span class="text-gray-500">${{ invoice.total.toFixed(2) }}</span>
          </div>
          <div class="flex justify-between text-sm mt-2">
            <span class="font-medium text-gray-700">Tax ({{ (0.15 * 100).toFixed(0) }}%):</span>
            <span class="text-gray-500">${{ (invoice.total*0.15).toFixed(2) }}</span>
          </div>
          <div class="flex justify-between text-lg font-bold mt-4">
            <span class="text-gray-700">Total:</span>
            <span class="text-gray-500">${{ (invoice.total*0.15 + invoice.total).toFixed(2) }}</span>
          </div>
        </div>

        <div class="mt-8 border-t border-gray-200 pt-6">
          <p class="text-sm text-gray-500">Thank you for your business!</p>
        </div>

        <div class="mt-8 flex justify-end">
          <button
              @click="printInvoice"
              class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium text-gray-700 rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
          >
            Print Invoice
          </button>
        </div>
      </div>
    </div>
  </div>
</template> 