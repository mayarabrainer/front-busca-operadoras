<script setup>
import { ref, computed } from "vue";
import "../assets/styles.css";

const searchName = ref("");
const operators = ref([]);
const searchCompleted = ref(false);
const loading = ref(false);

const currentPage = ref(1);
const itemsPerPage = 2;

const searchOperator = async () => {
  searchCompleted.value = false;
  operators.value = [];
  loading.value = true;
  currentPage.value = 1;

  try {
    const url = searchName.value.trim()
      ? `https://api-busca-operadoras.onrender.com/buscar_operadoras?q=${encodeURIComponent(searchName.value)}`
      : `https://api-busca-operadoras.onrender.com/buscar_operadoras`;

    const response = await fetch(url);

    if (!response.ok) {
      throw new Error("Error fetching operator.");
    }

    const data = await response.json();
    operators.value = data;
  } catch (error) {
    console.error("Error:", error);
    operators.value = [];
  } finally {
    loading.value = false;
    searchCompleted.value = true;
  }
};

const paginatedOperators = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  return operators.value.slice(start, start + itemsPerPage);
});

const totalPages = computed(() => Math.ceil(operators.value.length / itemsPerPage));

const nextPage = () => {
  if (currentPage.value < totalPages.value) currentPage.value++;
};

const prevPage = () => {
  if (currentPage.value > 1) currentPage.value--;
};
</script>

<template>
  <div class="app-container">
    <div class="search-container" data-aos="fade-down">
      <h1 class="title">Search Operator</h1>

      <div class="search-box">
        <input
          v-model="searchName"
          type="text"
          placeholder="Enter name or CNPJ"
          class="input-search"
        />
        <button
          @click="searchOperator"
          class="button-search"
          :disabled="loading"
        >
          {{ loading ? "Searching..." : "Search" }}
        </button>
      </div>

      <div v-if="loading" class="loading">
        Searching operators...
      </div>

      <div v-if="operators.length" class="result-box">
        <div v-for="(operator, index) in paginatedOperators" :key="index" class="operator-item" data-aos="zoom-in">
          <h2>{{ operator.Razao_Social }}</h2>
          <p><strong>CNPJ:</strong> {{ operator.CNPJ }}</p>
          <p><strong>City:</strong> {{ operator.Cidade }} - {{ operator.UF }}</p>
          <p><strong>Phone:</strong> {{ operator.Telefone }}</p>
          <p><strong>Email:</strong> {{ operator.Endereco_eletronico }}</p>
        </div>
      </div>

      <p v-if="searchCompleted && operators.length === 0" class="not-found">
        No operators found.
      </p>

      <div v-if="operators.length > 0" class="pagination">
        <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
      </div>
    </div>
  </div>
</template>
