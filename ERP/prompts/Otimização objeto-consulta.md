Vamos trabalhar na melhora da performance da classe DetalheAjusteGeradoPorRegraFiscalSaidaOrigem. 

Ao utiliza-la, estamos inicializando muitos objetos lazy, quando temos multiplas notas de entrada o cenário piora bastante. 

Vamos vencer ponto a ponto, primeiro vamos criar uma consulta nativa em NotaFiscalDao para substituir a condição da linha 63 `if (noneMatch(notasFiscaisDeSaidaOrigem, NotaFiscal::isNotaOuPedidoBaseadosEmRegraFiscal))`. 

Inspecione o método isNotaOuPedidoBaseadosEmRegraFiscal para entender os joins e filtros que devem ser aplicados para criar a query que trará um boolean. 

Para entender a estrutura das tabelas e nome das colunas e relacionamentos, inspecione erp-main-parent/bee/tables. 

Neste primeiro momento, não crie teste de integração nem de unidade.