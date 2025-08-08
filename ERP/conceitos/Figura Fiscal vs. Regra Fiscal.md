### Conceitos: Figura Fiscal vs. Regra Fiscal  
  
- Figura Fiscal (o “quando”): Representa o contexto tributário da operação. É o conjunto de condições que definem como tributar um cenário, por exemplo:  
  - Tipo de operação: entrada (compra) ou saída (venda).  
  - Finalidade: revenda, consumo/uso, industrialização, imobilizado, bonificação, demonstração etc.  
  - Localidade: intraestadual x interestadual; UF de origem e destino.  
  - Perfil dos envolvidos: sua filial e o fornecedor/cliente são contribuintes de ICMS? Qual o regime (Simples, Lucro Presumido/Real)?  
  - Outras características: com/sem ST, com/sem IPI, frete, transportadora, operação com FCP etc.  
  Pense nela como o “cenário de negócio” que seleciona qual lógica tributária aplicar.  
  
- Regra Fiscal (o “como”): Define a aplicação concreta dos tributos dentro de uma Figura Fiscal. Especifica CFOP, CST/CSOSN, alíquotas, bases e tratamentos para ICMS/FCP/ST, IPI, PIS/COFINS, reduções, créditos, etc. Pode ser:  
  - Específica por produto;  
  - Por NCM/CEST;  
  - Por grupo de mercadorias;  
  - Genérica (fallback) para cobrir lacunas.  
  
Em resumo: a Figura Fiscal seleciona o contexto; a Regra Fiscal, dentro desse contexto, define como tributar cada item.  
  
---  
  
### Por que aparece “Contém produtos com quantidade sem regra fiscal” ao fechar o pedido  
  
Ao fechar um Pedido de Compra (PC), o ERP precisa que cada item tenha uma Regra Fiscal aplicável. Sem regra, não há CFOP/CST/aliquotas para calcular os impostos de entrada, nem para gerar escrituração/contábil corretamente. A mensagem indica que:  
- Não há Regra Fiscal que cubra o produto/NCM/grupo no contexto daquela Figura; ou  
- Nem a Figura Fiscal compatível foi encontrada para o cenário; ou  
- Há regra, mas os filtros (UF, finalidade, contribuinte/regime, etc.) não batem; ou  
- A regra existe, mas está restrita a outra empresa/filial/grupo de produto.  
  
---  
  
### Relação com atores e módulos do ERP  
  
- ERP Comercial (Compras): define finalidade do pedido (revenda/consumo/imobilizado), escolhe fornecedor e itens. Essas escolhas formam o contexto para a Figura Fiscal.  
- ERP Fiscal: parametriza Figuras Fiscais (contextos) e Regras Fiscais (tributação dentro do contexto). Mantém CFOP, CST/CSOSN, alíquotas por UF/cenário.  
- ERP Core (Cadastros):  
  - Fornecedor: UF, inscrição estadual, regime tributário.  
  - Produto: NCM, CEST, tipo/grupo de mercadoria.  
  - Empresa/Filial: UF e regime tributário.  
- ERP Operação (Recebimento): valida e processa a entrada, usando as regras aplicadas.  
- ERP Financeiro/Contábil: consome os resultados para apuração e contabilização.  
  
Atores típicos:  
- Fiscal/Tributário: cria/ajusta Figuras e Regras Fiscais.  
- Compras: define a finalidade do pedido e aciona o Fiscal quando surgir “sem regra”.  
- Cadastro (Produtos/Fornecedores): garante NCM/CEST corretos e dados fiscais de fornecedores.  
- Operação/Recebimento: executa a entrada baseada nas regras.  
  
---  
  
### Passo a passo prático para resolver o erro no seu pedido  
  
1) Identifique o contexto do pedido:  
- Filial compradora (UF, regime tributário).  
- Fornecedor (UF, se é contribuinte de ICMS, regime).  
- Finalidade do pedido (revenda, consumo, imobilizado etc.).  
- Itens: NCM, CEST, grupo de mercadorias.  
  
2) Verifique se existe Figura Fiscal compatível (Entrada):  
- Deve cobrir UF origem x UF destino, finalidade, e perfil tributário dos participantes.  
- Se não existir, peça ao time Fiscal para criar.  
  
3) Dentro da Figura, confira as Regras Fiscais para cada item:  
- Cobertura por produto específico, NCM/CEST ou grupo do produto.  
- Definir CFOP de ENTRADA adequado (intra x inter), CST/CSOSN, ICMS (alíquota/base/redução/ST quando aplicável), IPI, PIS/COFINS, FCP etc.  
- Se faltar, criar regra específica ou por NCM/grupo. Avaliar se há necessidade de fallback.  
  
4) Reaplique a tributação no pedido:  
- Após incluir/ajustar regras, acione a função de “recalcular tributos/atualizar regras” no pedido.  
- Verifique se cada item recebeu CFOP/CST/aliquotas.  
  
5) Feche o pedido e prossiga com o recebimento:  
- Com as regras aplicadas, o fechamento não deve mais bloquear.  
  
6) Sobre NF-e exigir pedido:  
- O vínculo com PC é comum para garantir conferência quantitativa/financeira/fiscal.  
- Se houver fluxo excepcional de “entrada sem pedido”, confirme o procedimento com Fiscal/Operação e as regras necessárias.  
  
---  
  
### Exemplos ilustrativos de Figura/Regra  
  
- Compra para Revenda, Intraestadual, fornecedor contribuinte:  
  - Figura: Entrada | Revenda | Intraestadual | Fornecedor contribuinte.  
  - Regra: CFOP 1.xxx (p.ex. 1.102/1.949 conforme o item), CST/CSOSN apropriado, ICMS normal ou ST quando aplicável, PIS/COFINS conforme regime, IPI se devido.  
  
- Compra para Consumo, Interestadual:  
  - Figura: Entrada | Consumo | Interestadual | Fornecedor contribuinte.  
  - Regra: CFOP 2.xxx adequado, tratamento de DIFAL quando aplicável, PIS/COFINS, IPI conforme o item.  
  
- Compra com Substituição Tributária:  
  - Figura: Entrada | Revenda | Intra/Inter | Com ST.  
  - Regra: CFOP e CST de ST (p.ex. CST 60/10/70 conforme o caso), cálculo de ICMS-ST (MVA, base) e FCP-ST quando houver.  
  
Observação: Exemplos são genéricos; a parametrização final deve ser validada pelo Fiscal e pela legislação vigente.  
  
---  
  
### Checklist rápido para evitar “sem regra fiscal”  
  
- Produto: NCM/CEST corretos? Grupo de mercadoria definido?  
- Fornecedor: UF, IE e regime tributário preenchidos e válidos?  
- Filial: UF e regime tributário corretos?  
- Pedido: Finalidade correta (revenda/consumo/imobilizado)?  
- Fiscal: Existe Figura para o contexto? Há regra por produto/NCM/grupo para todos os itens? Há fallback (se a política permitir)?  
- Processo: Recalcular tributos no pedido após criar/ajustar regras.  
  
---  
  
### Quem faz o quê (papéis práticos)  
  
- Fiscal/Tributário: cria e mantém Figuras e Regras, homologa CFOP/CST/alíquotas.  
- Compras: define finalidade do pedido e aciona Fiscal diante de lacunas.  
- Cadastro: garante NCM/CEST e dados fiscais de fornecedores corretos.  
- Operação: realiza o recebimento apoiado nas regras aplicadas.  
  
---  
  
### Dicas úteis  
  
- Se o erro ocorre só em alguns itens, compare com itens similares que funcionam: pode faltar NCM/CEST ou o grupo está diferente do filtro da regra.  
- Mantenha ao menos uma cobertura por NCM/grupo para as Figuras mais críticas (revenda/consumo intra e inter).  
- Mudanças de regime tributário (da filial ou de fornecedores) exigem revisão das Figuras/Regras afetadas.  
  
---  
  
### Se quiser ajuda direcionada  
  
Compartilhe: filial, fornecedor, finalidade, UF origem/destino, NCM/CEST dos itens e se o fornecedor é contribuinte. Com isso, é possível indicar a Figura/Regra exata a criar/ajustar e sugerir CFOP/CST típicos para validação do Fiscal.  
  
---  
  
### Sugestão de melhoria na documentação interna  
  
- Criar um guia “Como diagnosticar ‘sem regra fiscal’” no módulo Fiscal, incluindo:  
  - Mapa de decisão de Figura Fiscal (entrada/saída, finalidade, UF, perfil contribuinte).  
  - Checklist de dados mínimos (produto, fornecedor, filial, finalidade).  
  - Exemplos de regras por NCM/grupo mais usados na operação.  
  - Procedimento para “recalcular tributos” no pedido após ajustes de regra.