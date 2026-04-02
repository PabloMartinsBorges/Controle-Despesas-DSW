// Importando as funções necessárias do Vue 3
const { createApp, ref, computed } = Vue;

const app = createApp({
  setup() {
    const transacoes = ref([]); 
    const novaDescricao = ref(''); 
    const novoValor = ref('');
    
    const paginaAtual = ref(1);
    const itensPorPagina = 5;


    const totalPaginas = computed(() => {
      return Math.ceil(transacoes.value.length / itensPorPagina);
    });

    const adicionarTransacao = () => {
      if (!novaDescricao.value || !novoValor.value) return;

      transacoes.value.push({
        descricao: novaDescricao.value,
        valor: parseFloat(novoValor.value)
      });

      // Limpa o formulário após adicionar
      novaDescricao.value = '';
      novoValor.value = '';
      
    };



  }
  
}).mount('#app'); // Monta a aplicação na div principal