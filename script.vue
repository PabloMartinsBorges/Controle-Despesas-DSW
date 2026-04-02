const { createApp, ref, computed } = Vue;

 
const TransacaoItem = {
  props: ['transacao', 'index', 'paginaAtual', 'totalPaginas', 'tamanhoPagina'],
  template: `
    <tr>
      <td>
        <button @click="$emit('mover', transacao.id, 'cima')" :disabled="paginaAtual === 1 && index === 0">up</button>
        <button @click="$emit('mover', transacao.id, 'baixo')" :disabled="paginaAtual === totalPaginas && index === tamanhoPagina - 1">down</button>
      </td>
      
      <td>{{ transacao.descricao }}</td>
      
      <td :class="{'credito': transacao.valor >= 0, 'debito': transacao.valor < 0}">
        R$ {{ transacao.valor.toFixed(2) }}
      </td>
      
      <td :class="{'saldo-negativo': transacao.saldo < 0}">
        R$ {{ transacao.saldo.toFixed(2) }}
      </td>
      
      <td>
        <button @click="$emit('remover', transacao.id)">Remover</button>
      </td>
    </tr>
  `
};


const app = createApp({
  setup() {
    const transacoes = ref([]); 
    const novaDescricao = ref(''); 
    const novoValor = ref('');
    
    const paginaAtual = ref(1);
    const itensPorPagina = 5; 

    const transacoesComSaldo = computed(() => {
      let saldoAcumulado = 0;
      return transacoes.value.map(transacao => {
        saldoAcumulado += transacao.valor;
        return { ...transacao, saldo: saldoAcumulado };
      });
    });

    const transacoesPaginadas = computed(() => {
      const inicio = (paginaAtual.value - 1) * itensPorPagina;
      const fim = inicio + itensPorPagina;
      return transacoesComSaldo.value.slice(inicio, fim);
    });

    const totalPaginas = computed(() => {
      return Math.ceil(transacoes.value.length / itensPorPagina) || 1;
    });

    const adicionarTransacao = () => {
      if (!novaDescricao.value || novoValor.value === '') return;

      transacoes.value.push({
        id: Date.now(),
        descricao: novaDescricao.value,
        valor: parseFloat(novoValor.value)
      });

      // Limpa o formulário após adicionar
      novaDescricao.value = '';
      novoValor.value = '';
    };

    const removerTransacao = (id) => {
      transacoes.value = transacoes.value.filter(t => t.id !== id);
      if (paginaAtual.value > totalPaginas.value && totalPaginas.value > 0) {
        paginaAtual.value = totalPaginas.value;
      }
    };


    const paginaAnterior = () => {
      if (paginaAtual.value > 1) paginaAtual.value--;
    };

    const proximaPagina = () => {
      if (paginaAtual.value < totalPaginas.value) paginaAtual.value++;
    };

    return {
      novaDescricao,
      novoValor,
      transacoesPaginadas,
      paginaAtual,
      totalPaginas,
      adicionarTransacao,
      removerTransacao,
      paginaAnterior,
      proximaPagina
    };
  }
});

app.mount('#app'); // Monta a aplicação na div principal