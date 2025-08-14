 
A lógica de vinculação (matching) fica na classe NotaFiscalEletronicaToNotaFiscalConverter e ocorre em camadas, seguindo uma ordem de prioridade. Em resumo, o sistema tenta vincular nesta ordem:  
  
1) GTIN/EAN do XML  
- Campos usados: prod.CEAN e prod.CEANTrib.  
- O sistema carrega previamente um mapa de produtos por EAN (barcodes) e tenta achar o produto do ERP pelo EAN informado no XML.  
- Observação: valores “0”/vazios são descartados.  
  
2) cProd como GTIN (se configurado)  
- Parâmetro: ConfiguracaoIntegracaoComercial.isConsiderarCodigoProdutoComoGtin().  
- Quando ativo, o sistema também tenta interpretar o cProd do XML como se fosse um GTIN e busca por EAN usando esse valor.  
  
3) Referência de faturamento em devolução (se configurado)  
- Escopo: Notas de devolução de entrada.  
- Parâmetro: Parametro.IMPORTACAO_DEVOLUCAO_CONSIDERAR_CODIGO_REFERENCIA_FATURAMENTO_PRODUTO.  
- Se habilitado, o sistema usa o cProd do XML para buscar produtos pela “referência de faturamento” cadastrada no ERP.  
- Se encontrar mais de um produto com a mesma referência, o sistema registra log de duplicidade e não escolhe um arbitrariamente (precisa intervenção).  
  
4) cProd como código interno do produto (somente em casos específicos)  
- Variável de controle: notaFiscalService.deveCarregarPorCodigoSeNaoTiverGtin(notaFiscal).  
- Essa checagem retorna true para situações como:  
  - Vasilhame/sacaria entre lojas;  
  - Devoluções de compra quando ConfiguracaoIntegracaoComercial.isConsiderarCodigoProdutoNaImportacaoDeDevolucoes() estiver ativo;  
  - Notas de importação e NF de importação com extrato DI.  
- Se verdadeiro e o cProd for numérico > 0, o sistema tenta tratar o cProd como o próprio produtoKey interno do ERP e vincula por esse código.  
  
5) Código de referência do fornecedor (cProd) e variações  
- O sistema carrega o cadastro de produtos do fornecedor (incluindo matriz/filiais) e mapeia:  
  - FornecedorProduto.codigoReferencia (código principal) e  
  - FornecedorProduto.codigosReferenciaSecundario (códigos adicionais).  
- Na hora de vincular, tenta nesta ordem:  
  - Exato com zeros/prefixos: produtosDoCadastroPorCodigoDeReferenciaDoFornecedorSemTirarZerosAEsquerda.get(cProd)  
  - Exato “sem zeros/normalizado”: produtosDoCadastroPorCodigoDeReferenciaDoFornecedor.get(cProd)  
  - Normalizado (alfanumérico sem zeros à esquerda): produtosDoCadastroPorCodigoDeReferenciaDoFornecedor.get(normalizar(cProd))  
- A normalização mencionada remove zeros à esquerda e caracteres não alfanuméricos conforme utilitário interno.  
  
6) Estratégia especial para Operação Entre Lojas  
- Quando a nota é de operação entre lojas, há uma estratégia específica (factory/strategy) que pode levar em conta o parâmetro “considerar cProd como GTIN” e outros detalhes do cenário entre lojas, e ainda faz um fallback para referência de faturamento.  
  
Importante: Nome/xProd NÃO entra na priorização de matching  
- O campo xProd (descrição do produto no XML) não é utilizado para localizar o produto no ERP. Ele aparece apenas em mensagens/logs quando o produto não é encontrado.  
  
---  
  
### Parâmetros e configurações que influenciam o de-para  
  
- Considerar cProd como GTIN na busca por EAN  
  - API: ConfiguracaoIntegracaoComercial.isConsiderarCodigoProdutoComoGtin().  
  - Efeito: Permite que cProd seja usado como GTIN/EAN (além de CEAN/CEANTrib).  
  
- Considerar cProd como referência de faturamento em devolução de entrada  
  - Parâmetro de sistema: Parametro.IMPORTACAO_DEVOLUCAO_CONSIDERAR_CODIGO_REFERENCIA_FATURAMENTO_PRODUTO.  
  - Efeito: Em devoluções de entrada, usa cProd para buscar por “referência de faturamento”.  
  
- Tentar usar cProd como código interno (produtoKey) quando não houver GTIN  
  - Método: NotaFiscalService.deveCarregarPorCodigoSeNaoTiverGtin(notaFiscal).  
  - Depende de:  
    - Vasilhame/sacaria entre lojas; ou  
    - Devolução de compra com ConfiguracaoIntegracaoComercial.isConsiderarCodigoProdutoNaImportacaoDeDevolucoes() ativo; ou  
    - Notas de importação; ou NF de importação com extrato DI.  
  
- Validação de duplicidade de código de referência do fornecedor  
  - Parâmetro de sistema: Parametro.VALIDAR_CODIGO_DE_REFERENCIA_DUPLICADO_NO_FORNECEDOR.  
  - Efeito: Se houver o mesmo código de referência de fornecedor em mais de um produto, o sistema registra log de ERRO (quando ativo) ou AVISO (quando desativado). Essa validação roda após o vínculo dos itens.  
  
---  
  
### Resposta direta às suas perguntas  
  
- É por nome/xProd? Não. O nome (xProd) não é utilizado na lógica de vínculo, apenas em logs/avisos.  
- É por cEAN? Sim. Primeira prioridade é por GTIN/EAN: CEAN e CEANTrib do XML.  
- É por cProd? Sim, mas de formas diferentes conforme configuração e contexto:  
  - Como GTIN (se “considerar cProd como GTIN” estiver ativo);  
  - Como Referência de Faturamento (em devoluções com parâmetro ativo);  
  - Como código interno (produtoKey) em cenários específicos (sem GTIN e nos casos cobertos por deveCarregarPorCodigoSeNaoTiverGtin).  
  - Como código de referência do fornecedor (incluindo códigos secundários), com variações de normalização (com/sem zeros à esquerda).  
- Existe um parâmetro que define a escala de prioridade? Não há um único parâmetro que “ordene” a escala. A ordem é definida no código (conforme listado acima), mas alguns parâmetros mudam quais etapas entram no jogo (por exemplo, habilitar cProd como GTIN, habilitar referência de faturamento em devolução ou permitir usar cProd como código interno em certos cenários).  
  
---  
  
### Dicas práticas para evitar problemas de vínculo  
  
- Garanta que os GTINs (CEAN/CEANTrib) estejam corretos no XML e no cadastro do produto (idealmente é a forma mais estável).  
- Se o fornecedor usa códigos próprios (cProd), cadastre-os em FornecedorProduto (código principal e secundários). O sistema tentará casar exatamente, depois sem zeros à esquerda e versão normalizada.  
- Em devoluções, se deseja usar a referência de faturamento via cProd, habilite IMPORTACAO_DEVOLUCAO_CONSIDERAR_CODIGO_REFERENCIA_FATURAMENTO_PRODUTO.  
- Se o fornecedor manda cProd como GTIN, avalie ativar “considerar código do produto como GTIN” na integração comercial.  
- Evite duplicidade do mesmo código de referência em múltiplos produtos do fornecedor; caso inevitável, saiba que o sistema registra log e pode impedir o vínculo automático.  
  
---  
  
### Onde isso acontece no código (para sua conferência)  
  
- Carregamento e prioridade do de-para: NotaFiscalEletronicaToNotaFiscalConverter.obterProdutoDoCadastro(...).  
- Carregamento dos mapas de referência: métodos carregarProdutosPorEAN, carregarProdutosPorEANConsiderandoParametroCodigoProdutoComoGtin, carregarProdutosPorCodigoQuandoNaoTiverGtin, carregarProdutosPorCodigoReferenciaFaturamentoQuandoNaoTiverGtin, carregarProdutosDoFornecedor.  
- Casos para usar cProd como código interno: NotaFiscalService.deveCarregarPorCodigoSeNaoTiverGtin(...).  
- Validação de duplicidade: gerarLogDeCodigoDeReferenciaDuplicado e uso do parâmetro VALIDAR_CODIGO_DE_REFERENCIA_DUPLICADO_NO_FORNECEDOR.  
  
---  
  
### Possíveis correções quando o vínculo falha  
  
- Conferir se o EAN do XML consta no cadastro (produto e embalagem);  
- Conferir se os códigos do fornecedor (cProd) estão cadastrados como referência principal ou secundária no FornecedorProduto;  
- Ajustar os parâmetros de integração comercial se o fornecedor envia cProd como GTIN, ou se quer usar cProd em devoluções;  
- Resolver duplicidades de códigos de referência do fornecedor.  