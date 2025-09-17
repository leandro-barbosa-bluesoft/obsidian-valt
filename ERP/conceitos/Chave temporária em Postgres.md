### Contexto e causa do erro  
- No ERP, a tabela `keys` é usada como uma “tabela de trabalho” para carregar coleções de IDs e reutilizá-los em SQLs/procedures. Ex.: `ProdutoDao.incluirInTableKeys` insere em `keys` via `insert into keys(keyi1) ...`.  
- Em Oracle essa tabela existe (normalmente como GTT – Global Temporary Table). No PostgreSQL ela não está criada por padrão como tabela permanente: usamos uma tabela temporária criada na sessão.  
- Existe um `dbchange` justamente para isso: `erp-main-parent/bee/dbchanges/1656633274987-debt-criando-todas-tabelas-temporarias-do-oracle-no-postgres.dbchange`. Ele cria a procedure `utils.create_temp()` que, quando executada, cria várias tabelas temporárias, incluindo `keys`:  
  - DDL relevante: `create temporary table if not exists keys (...) on commit delete rows;`  
- A procedure do relatório `erp-main-parent/bee/procedures/relatorio_cobertura.bee` (bloco `text_postgres`) insere e lê de `keys`. Se você executá-la em uma sessão onde `keys` ainda não foi criada, o PostgreSQL retorna `ERROR: relation "keys" does not exist` (seu erro).  
  
### A solução recomendada (PostgreSQL)  
Crie a tabela temporária `keys` na sessão antes de chamar o relatório. Não crie manualmente toda vez: use a procedure utilitária prevista pelo projeto.  
  
Passos:  
1) Em sua sessão/conexão do PostgreSQL, execute:  
```  
call utils.create_temp();  
```  
- Isso criará (na sessão) todas as temp tables padronizadas do ERP, incluindo `keys`.  
- Obs.: é “por sessão”. Se você fechar a conexão, terá que executar novamente na nova sessão.  
  
2) Em seguida, execute a procedure do relatório:  
```  
call public.relatorio_cobertura();  
```  
- O schema é `public` conforme o arquivo da procedure (`text_postgres`).  
  
3) Se precisar reexecutar dentro da mesma sessão, recomenda-se limpar a `keys` (e/ou confiar que a própria procedure usa `truncate`/`on commit delete rows` conforme o fluxo):  
```  
delete from keys;  
```  
- Como `keys` é `on commit delete rows`, um `COMMIT` também esvazia a tabela.  
  
### E se `utils.create_temp()` não existir na base?  
Em ambientes onde o `dbchange` ainda não foi aplicado, você tem duas alternativas temporárias:  
- Aplicar o `dbchange` oficial (recomendado em ambientes gerenciados pelo ERP).  
- Criar apenas a `keys` mínima na sessão atual (quebra-galho para rodar o relatório):  
```  
create temporary table if not exists keys (  
  keyi1 int,  keyi2 int,  keyi3 int,  keys1 varchar(400),  keys2 varchar(400),  keys3 varchar(400),  keyi4 int,  keyi5 int,  keyi6 int,  keyd1 date,  keyi7 int,  keyd2 date,  keyd3 date,  keyi8 int,  keyi9 int) on commit delete rows;  
create index if not exists idx_keys on keys(keyi1,keyi2,keyi3);  
```  
- Para o `relatorio_cobertura`, os campos usados são `keyi1`, `keyi2` e `keyi3`. Os demais são para compatibilidade com outros usos no ERP.  
  
### Dicas práticas  
- No IntelliJ/IDEA, certifique-se de estar executando no console do PostgreSQL (não no de Oracle). O prompt do seu log mostra `oracle> call relatorio_cobertura()`. No PostgreSQL o correto é `call public.relatorio_cobertura();`. Se estiver no console Oracle, ele tentará rodar a versão Oracle e seu back-end PostgreSQL não reconhecerá a tabela.  
- Sempre rode `call utils.create_temp();` após abrir a conexão do console (uma vez por sessão) antes de usar procedures que dependem de temp tables, como `keys`.  
  
### Referências no código do projeto  
- Criação das temps no PostgreSQL: `erp-main-parent/bee/dbchanges/1656633274987-debt-criando-todas-tabelas-temporarias-do-oracle-no-postgres.dbchange` (cria `utils.create_temp()` e a `keys` como temp table).  
- Uso da `keys` no relatório: `erp-main-parent/bee/procedures/relatorio_cobertura.bee` (blocos `text` e `text_postgres`).  
- Uso típico em DAO: `erp-comercial-parent/.../ProdutoDao.java` método `incluirInTableKeys`, que faz `insert into keys(keyi1) values (:produtoKey)`.  
  
### Resposta objetiva  
Sim, a `keys` é tratada como tabela temporária no PostgreSQL. A melhor solução é criar a `keys` temporariamente na sessão antes de rodar a procedure, usando a procedure utilitária do ERP:  
- Execute `call utils.create_temp();`  
- Depois `call public.relatorio_cobertura();`  
Caso `utils.create_temp()` não exista na sua base, crie a temp table `keys` manualmente (DDL acima) como workaround e, idealmente, aplique o `dbchange` correspondente.  
  
### Sugestão de melhoria da guideline  
- Incluir nos guias de execução de relatórios/procedures em PostgreSQL uma observação padrão: “Antes de executar procedures que usam tabelas temporárias herdadas do Oracle (ex.: `keys`), executar `call utils.create_temp();` na sessão.”