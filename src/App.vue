<template>
  <Header />
  <div class="container">
    <Balance :total="total" />
    <IncomeExpenses :income="income" :expenses="expenses" />
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
    <AddTransaction @transactionAdded="handleTransactionSubmitted" />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useToast } from 'vue-toastification';

import Header from '@/components/Header.vue';
import Balance from './components/Balance.vue';
import IncomeExpenses from './components/IncomeExpenses.vue';
import TransactionList from './components/TransactionList.vue';
import AddTransaction from './components/AddTransaction.vue';

const toast = useToast();

const transactions = ref([]);

onMounted(() => {
  const savedTransactions = JSON.parse(localStorage.getItem('transactions'));

  if (savedTransactions) {
    transactions.value = savedTransactions;
  }
});

const total = computed(() => {
  return transactions.value.reduce((total, transaction) => {
    return total + transaction.amount;
  }, 0);
});

const income = computed(() => {
  return transactions.value.reduce((total, transaction) => {
    return total + (transaction.amount > 0 ? transaction.amount : 0);
  }, 0);
});

const expenses = computed(() => {
  return transactions.value.reduce((total, transaction) => {
    return total + (transaction.amount < 0 ? transaction.amount : 0);
  }, 0);
});

const handleTransactionSubmitted = (transaction) => {
  transactions.value.push({
    id: generateUniqueId(),
    ...transaction
  });

  toast.success('Transaction added');
  savedTransactions()
};

const generateUniqueId = () => {
  return Math.floor(Math.random() * 1000000);
}

const handleTransactionDeleted = (id) => {
  transactions.value = transactions.value.filter((transaction) => transaction.id !== id);

  toast.success('Transaction deleted');
  savedTransactions()
}

const savedTransactions = () => {
  localStorage.setItem('transactions', JSON.stringify(transactions.value));
};
</script>