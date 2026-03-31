// Importando as funções necessárias do Vue 3
const { createApp, ref, computed } = Vue;

const app = createApp({
  setup() {
    const transacoes = ref([]); 
    const novaDescricao = ref(''); 
    const novoValor = ref('');
    
    const paginaAtual = ref(1);
    const itensPorPagina = 5;

  }
}).mount('#app'); // Monta a aplicação na div principal