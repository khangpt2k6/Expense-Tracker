<template>
  <div class="page-wrapper">
    <Header />
    <div class="container">
      <Balance 
        :total="total" 
        :animateBalance="animateBalance" 
      />
      <IncomeExpenses 
        :income="+income" 
        :expenses="+expenses" 
        :animateValues="animateValues"
      />
      <TransactionList
        :transactions="transactions"
        @transactionDeleted="handleTransactionDeleted"
      />
      <AddTransaction 
        @transactionSubmitted="handleTransactionSubmitted" 
      />
    </div>
  </div>
</template>

<script setup>
import Header from './components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

import { ref, computed, onMounted, watch } from 'vue';
import { useToast } from 'vue-toastification';


const toast = useToast();
const transactions = ref([]);
const animateBalance = ref(false);
const animateValues = ref(false);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));
  
  if (savedTransactions) {
    // Add a slight delay for the initial load animation
    setTimeout(() => {
      transactions.value = savedTransactions;
    }, 300);
  }
});

// Get total
const total = computed(() => {
  return transactions.value.reduce((acc, transaction) => {
    return acc + transaction.amount;
  }, 0);
});

// Get income
const income = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => acc + transaction.amount, 0)
    .toFixed(2);
});

// Get expenses
const expenses = computed(() => {
  return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => acc + transaction.amount, 0)
    .toFixed(2);
});

// Watch for changes to trigger animations
watch(total, () => {
  animateBalance.value = true;
  animateValues.value = true;
  
  setTimeout(() => {
    animateBalance.value = false;
    animateValues.value = false;
  }, 600);
});

// Submit transaction with explosive feedback
const handleTransactionSubmitted = (transactionData) => {
  // Create new transaction object
  const newTransaction = {
    id: generateUniqueId(),
    text: transactionData.text,
    amount: transactionData.amount,
  };
  
  // Add transaction with a small delay for visual effect
  setTimeout(() => {
    transactions.value.push(newTransaction);
    saveTransactionsToLocalStorage();
    
    // Show different toast messages based on transaction type
    if (transactionData.amount > 0) {
      toast.success('💰 Income added successfully!', {
        icon: '💸'
      });
    } else {
      toast.info('💸 Expense recorded!', {
        icon: '📝'
      });
    }
  }, 100);
};

// Generate unique ID with timestamp for better uniqueness
const generateUniqueId = () => {
  return Date.now() + Math.floor(Math.random() * 1000000);
};

// Delete transaction with explosive feedback
const handleTransactionDeleted = (id) => {
  // Find the transaction before deleting it
  const transactionToDelete = transactions.value.find(
    transaction => transaction.id === id
  );
  
  // Delete the transaction
  transactions.value = transactions.value.filter(
    (transaction) => transaction.id !== id
  );
  
  saveTransactionsToLocalStorage();
  
  // Show appropriate toast based on transaction type
  if (transactionToDelete.amount > 0) {
    toast.warning('Income transaction removed!', {
      icon: '🗑️'
    });
  } else {
    toast.success('Expense transaction removed!', {
      icon: '✨'
    });
  }
};

// Save transactions to local storage
const saveTransactionsToLocalStorage = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
};
</script>

<style scoped>
.page-wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>