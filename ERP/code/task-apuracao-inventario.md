Configurar no VM Options:

`-Ddatasource.primary.url="jdbc:oracle:thin:@aws-db1-us-test.cd6uowqooguq.us-east-1.rds.amazonaws.com:1521:test"`
`-Ddatasource.primary.username="XZmiRFTy[pedreira]"`
`-Ddatasource.primary.password="BLpxFTJPvpWWbaja"`

`-Ddatasource.secondary.url="jdbc:redshift://pedreira.dw.bluesoft.com.br:5432/pedreira"`
`-Ddatasource.secondary.username="dev"`
`-Ddatasource.secondary.password="Bluesoft8437"`

`-Ddatasource.tertiary.url="jdbc:postgresql://aws-db-pg2:5432/pedreira"`
`-Ddatasource.tertiary.username="resumo_dev"`
`-Ddatasource.tertiary.password="resumo_dev"`

# Deploy
Atualizar manualmente no pom.xml a versão
Entrar em https://build.bluesoft.com.br/view/tornado/job/tornado-task-deploy/ para executar o build da task