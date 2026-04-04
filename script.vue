const { createApp, ref, computed } = Vue;

// COMPONENTE: Representa uma linha da tabela de transações 
const TransacaoItem = {
  // São os dados que o componente recebe da tabela principal
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

// app
const app = createApp({
  setup() {
    // ref() cria variáveis reativas
    const transacoes = ref([]); 
    const novaDescricao = ref(''); 
    const novoValor = ref('');
    
    const paginaAtual = ref(1);
    const itensPorPagina = 5;


    // computed() cria variáveis derivadas
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

      // Limpa o formulário após adicionar a transação e vai para a última página
      novaDescricao.value = '';
      novoValor.value = '';
      paginaAtual.value = totalPaginas.value;
    };

    const removerTransacao = (id) => {
      transacoes.value = transacoes.value.filter(t => t.id !== id);
      if (paginaAtual.value > totalPaginas.value && totalPaginas.value > 0) {
        paginaAtual.value = totalPaginas.value;
      }
    };

    // Altera a posição do item no array original
    const moverTransacao = (id, direcao) => {
      // Encontra a transação na lista
      const index = transacoes.value.findIndex(t => t.id === id);
      
      if (direcao === 'cima' && index > 0) {
        const temp = transacoes.value[index];
        transacoes.value[index] = transacoes.value[index - 1];
        transacoes.value[index - 1] = temp;
      } else if (direcao === 'baixo' && index < transacoes.value.length - 1) {
        const temp = transacoes.value[index];
        transacoes.value[index] = transacoes.value[index + 1];
        transacoes.value[index + 1] = temp;
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
      moverTransacao,
      paginaAnterior,
      proximaPagina
    };
  }
});

// Registra o componente 'TransacaoItem' para que o HTML o reconheça
app.component('transacao-item', TransacaoItem);
app.mount('#app'); // Monta a aplicação na div principal