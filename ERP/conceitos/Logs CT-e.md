### Visão geral

Você perguntou como o processamento de CT-e registra os erros que aparecem nas colunas “Status de Processamento” e “Status do CT-e” e no modal de erros da tela de Importação de CT-e. Abaixo explico de ponta a ponta onde esses dados são criados, como são persistidos (sim, como entidades), e como chegam à UI.

### Onde a tela busca os dados

- O grid da tela chama o endpoint GET `/erp-app/areas/estoques/conhecimento-transporte/importacao-cte/lista-arquivo-ctes`.
- O controller `ProcessadorNotaFiscalCteController.listarCte` retorna `List<IntegracaoCteView>` (arquivo: `.../web/controller/ProcessadorNotaFiscalCteController.java`).
- `IntegracaoCteView` é construído a partir da entidade de domínio `ArquivoRecebidoCte` (arquivo: `.../web/view/IntegracaoCteView.java`).

### O que são os “erros” mostrados no grid/modal

- Na UI, `row.entity.erros` (usado para pintar classe `text-danger` e no modal) vem de `IntegracaoCteView.getErros()`.
- `IntegracaoCteView` popula `this.erros = ErroIntegracaoCteView.toList(arquivoRecebidoCte.getLogs())`.
- `ErroIntegracaoCteView` é um DTO com `descricao`, que lê `ocorrencia` de cada `LogArquivoRecebidoCte`.
- Portanto, os “erros” são entidades persistidas: `LogArquivoRecebidoCte` (JPA) vinculadas a `ArquivoRecebidoCte`.
    - Classe: `br.com.bluesoft.tec.model.estoquenfe.conhecimentodetransporte.importarcte.LogArquivoRecebidoCte`.
    - Campos: referência ao `ArquivoRecebidoCte`, `tipoGravidadeLog` (para diferenciar erro), `tipoLogArquivoCteRecebido`, `ocorrencia` (a mensagem que a UI mostra).
    - Método `logErro()` retorna `true` quando a gravidade é de erro.

### Como e quando os erros são criados

A criação e persistência dos erros é feita por serviços de domínio que, ao validar/processar o CT-e, registram logs no `ArquivoRecebidoCte` e persistem no banco:

- Serviço central de logs: `ArquivoRecebidoCteLogService` (`.../domain/business/service/ArquivoRecebidoCteLogService.java`)
    
    - `gerarLogs(ArquivoRecebidoCte arquivo)`: validações como “remetente inexistente/transportadora não cadastrada”. Se falha:
        - Cria `LogArquivoRecebidoCte` via `LogArquivoRecebidoCteFactory.criar(arquivo, ocorrencia)` com `TipoGravidadeLog = ERRO` e `TipoLogArquivoCteRecebido = OUTROS`.
        - Marca arquivo como FALHOU: `arquivo.marcarArquivoComoFalhou()` (muda `TipoStatusArquivo` para FALHOU).
        - Adiciona log: `arquivo.addLog(log)`.
        - Persiste: `arquivoRecebidoCteDao.salvar(arquivo)`.
    - `gerarLogs(ArquivoRecebidoCte arquivo, Exception e)`: captura exceções (inclusive de regras) e grava ocorrência padronizada “Não foi possível processar o arquivo. Motivo: {mensagem}”, além de marcar FALHOU e salvar.
- Processo de ingestão do XML: `ArquivoRecebidoCteProcessadorService.processarCte` (`.../service/ArquivoRecebidoCteProcessadorService.java`)
    
    - Constrói/popula `ArquivoRecebidoCte` a partir do XML (via `ArquivoRecebidoCteFactory`), limpa logs antigos, salva o arquivo e chama `arquivoRecebidoCteLogService.gerarLogs(arquivo)` para validações iniciais (ex.: CNPJ de remetente não cadastrado), persistindo os logs de erro quando existirem.
- Verificação de status na SEFAZ e tratamento de exceções: `ProcessorNotaFiscalCteFacade` (`.../facade/ProcessorNotaFiscalCteFacade.java`)
    
    - `isStatusCTeValidoParaProcessar(arquivo)`: se status for CANCELADO, define `TipoStatusArquivo` como FALHOU, salva e chama `arquivoRecebidoCteLogService.gerarLogs(arquivo, new BusinessException("Status CT-e cancelado na SEFAZ."))`.
    - Se ocorre erro durante consulta à SEFAZ, seta `statusCte` para `null`, gera log com ocorrência “Ocorreu um erro ao consultar o CT-e na SEFAZ: {detalhe}”, e retorna não processável.
    - Ao final de um processamento bem-sucedido e com nota criada/confirmada, executa limpeza dos logs de erro e marca como `PROCESSADO_COM_SUCESSO`:
        - `arquivo.limparLogsDeErrosDeArquivosProcessadosComSucesso()`
        - `arquivoRecebidoCteProcessadorService.limparLogs(arquivo)` (limpa na base)
        - `arquivoRecebidoCteProcessadorService.atualizarArquivoRecebidoCteParaProcessado(arquivo)`
- Filtragem de erros no próprio domínio:
    
    - `ArquivoRecebidoCte.getLogsDeErros()` retorna apenas logs com `tipoGravidadeLog.isErro()`.
    - `ArquivoRecebidoCte.permiteReprocessar()` retorna true quando `TipoStatusArquivo == FALHOU`.

Conclusão: os erros exibidos são registros persistidos (entidade `LogArquivoRecebidoCte`) associados ao `ArquivoRecebidoCte`, criados tanto por validações de negócio quanto por tratamento de exceções, e salvos via DAO. A UI apenas mapeia esses logs para `ErroIntegracaoCteView` e exibe suas descrições.

### Como “Status de Processamento” e “Status do CT-e” são preenchidos

- “Status de Processamento” vem de `ArquivoRecebidoCte.getTipoStatusArquivo().getDescricao()` e é exibido em `IntegracaoCteView.status`.
- “Status do CT-e” vem de `ArquivoRecebidoCte.getStatusCte()` (enum `StatusArquivoRecebidoCteEnum`) e é exibido em `IntegracaoCteView.statusCte` (ou “-” se nulo). Esse status pode ser definido/atualizado pela consulta na SEFAZ realizada no `ProcessorNotaFiscalCteFacade`.

### Como isso impacta a UI

- No grid, as colunas “Status de Processamento” e “Status CT-e” ficam com classe `text-danger` quando `row.entity.erros.length > 0`.
- O modal “Visualizar Erros” lista `vm.nota.erros` com `{{ erro.descricao }}` — que vem diretamente de `LogArquivoRecebidoCte.ocorrencia`.

### Em resumo

- São entidades? Sim. Os “erros” são `LogArquivoRecebidoCte`, entidade JPA persistida e vinculada ao `ArquivoRecebidoCte`.
- Como são salvos? Via `ArquivoRecebidoCteLogService` + `LogArquivoRecebidoCteFactory`, anexados ao arquivo e persistidos com `ArquivoRecebidoCteDao.salvar`.
- É por exception? Também. Exceptions nas fases de processamento/consulta SEFAZ/validações são convertidas em logs de erro com mensagens padronizadas.
- Insert em log? Sim, os logs são tabelas de domínio (não apenas log textual), e são carregados na tela via `IntegracaoCteView`/`ErroIntegracaoCteView`.