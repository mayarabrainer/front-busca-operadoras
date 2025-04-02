<script setup>
import { ref, computed } from "vue";
import "../assets/styles.css";

const nomeBusca = ref("");
const operadoras = ref([]);
const buscaFinalizada = ref(false);
const carregando = ref(false);


const paginaAtual = ref(1);
const itensPorPagina = 2;

const buscarOperadora = async () => {
  buscaFinalizada.value = false;
  operadoras.value = [];
  carregando.value = true;
  paginaAtual.value = 1; 

  try {
    const url = nomeBusca.value.trim()
      ? `https://api-busca-operadoras.onrender.com/buscar_operadoras?q=${encodeURIComponent(nomeBusca.value)}`
      : `https://api-busca-operadoras.onrender.com/buscar_operadoras`;

    const response = await fetch(url);

    if (!response.ok) {
      throw new Error("Erro ao buscar operadora.");
    }

    const data = await response.json();
    operadoras.value = data;
  } catch (error) {
    console.error("Erro:", error);
    operadoras.value = [];
  } finally {
    carregando.value = false;
    buscaFinalizada.value = true;
  }
};


const operadorasPaginadas = computed(() => {
  const inicio = (paginaAtual.value - 1) * itensPorPagina;
  return operadoras.value.slice(inicio, inicio + itensPorPagina);
});


const totalPaginas = computed(() => Math.ceil(operadoras.value.length / itensPorPagina));


const avancarPagina = () => {
  if (paginaAtual.value < totalPaginas.value) paginaAtual.value++;
};

const voltarPagina = () => {
  if (paginaAtual.value > 1) paginaAtual.value--;
};
</script>

<template>
  <div class="app-container">
    <div class="search-container" data-aos="fade-down">
      <h1 class="title">Buscar Operadora</h1>

      <div class="search-box">
        <input
          v-model="nomeBusca"
          type="text"
          placeholder="Digite o nome ou CNPJ"
          class="input-search"
        />
        <button
          @click="buscarOperadora"
          class="button-search"
          :disabled="carregando"
        >
          {{ carregando ? "Buscando..." : "Buscar" }}
        </button>
      </div>

      <div v-if="carregando" class="loading">
        Buscando operadoras...
      </div>

      <div v-if="operadoras.length" class="result-box" >
        <div v-for="(operadora, index) in operadorasPaginadas" :key="index" class="operadora-item" data-aos="zoom-in">
          <h2>{{ operadora.Razao_Social }}</h2>
          <p><strong>CNPJ:</strong> {{ operadora.CNPJ }}</p>
          <p><strong>Cidade:</strong> {{ operadora.Cidade }} - {{ operadora.UF }}</p>
          <p><strong>Telefone:</strong> {{ operadora.Telefone }}</p>
          <p><strong>Email:</strong> {{ operadora.Endereco_eletronico }}</p>
        </div>
      </div>

  
      <p v-if="buscaFinalizada && operadoras.length === 0" class="not-found">
        Nenhuma operadora encontrada.
      </p>

      
      <div v-if="operadoras.length > 0" class="pagination">
        <button @click="voltarPagina" :disabled="paginaAtual === 1">Anterior</button>
        <span>Página {{ paginaAtual }} de {{ totalPaginas }}</span>
        <button @click="avancarPagina" :disabled="paginaAtual === totalPaginas">Próxima</button>
      </div>
    </div>
  </div>
</template>
