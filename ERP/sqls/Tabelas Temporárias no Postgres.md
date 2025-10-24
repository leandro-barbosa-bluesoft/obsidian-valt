No Postgres as tabelas temporárias tem escopo de conexão, ou seja, quando a conexão é fechada todas as tabelas são removidas.

Para resolver isso, foi criada uma procedure chamada **create_temp** que fica dentro do package **utils** e é chamada dentro do ERP sempre que uma nova conexão é aberta.

A chamada é feita assim `call utils.create_temp()`

Dentro dessa procedure existe a criação de todas as tabelas temporárias atuais do ERP.

No entanto, se for necessário criar uma nova tabela temporária, é necessário incluí-la manualmente dentro da procedure.

Para isso, crie um dbchange fazendo replace da procedure **create_temp**, e dentro do corpo da procedure você deve incluir a criação da sua tabela temporária seguindo a sintaxe abaixo:


`create temporary table if not exists nome_da_tabela_temporaria_exemplo1 (`
`coluna1 int,`
`coluna2 int,`
`coluna3 decimal(10, 4)`
`) on commit delete rows;`

`create temporary table if not exists nome_da_tabela_temporaria_exemplo2 (`
`coluna1 int,`
`coluna2 int,`
`coluna3 decimal(10, 4),`
`constraint nome_da_pk primary key (coluna1)`
`) on commit delete rows;`

`create unique index if not exists nome_do_indice on nome_da_tabela(coluna1,coluna2);`

Após criar seu dbchange, você deve re-gerar o schema da procedure. Para isso inicie o seu postgres local e na pasta **erp-main-parent** execute o seguinte comando:

`bee schema:generate pgsql create_temp`

E para validar todos os arquivos:

`bee schema:validate pgsql`

> No dbchange **1656633274987-debt-criando-todas-tabelas-temporarias-do-oracle-no-postgres.dbchange** existe um exemplo de como fazer.

[Wiki Bluesoft](https://wiki.bluesoft.com.br/books/engenharia-erp/page/padroes-multi-banco/edit?content-id=bkmrk-para-mais-detalhes-v&content-text=Para%20mais%20detalhes%20voc%C3%AA%20pode%20acessar%20o%20commit%20no%20g "Ir para a seção do editor")

Para mais detalhes você pode acessar o [commit no github](https://github.com/bluesoft/erp/commit/903f29206ff7196daa22c8928a12f17154aff88b)