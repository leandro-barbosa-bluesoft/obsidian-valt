tem que configurar o datasource jdbc/tec-resumo:

``<Resource name="jdbc/tec-resumo" auth="Container" type="javax.sql.DataSource"
                  driverClassName="org.postgresql.Driver"
                  factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
                  removeAbandoned="true" removeAbandonedTimeout="1200" logAbandoned="true"
                  testOnBorrow="true" validationQuery="select 1"
                  jdbcInterceptors="org.apache.tomcat.jdbc.pool.interceptor.StatementFinalizer"
                  initialSize="0" maxActive="4" minIdle="0" maxIdle="4"
                  defaultAutoCommit="false" rollbackOnReturn="true" defaultTransactionIsolation="READ_COMMITTED" readOnly="true"
                  url="jdbc:postgresql://aws-db-pg2:5432/beta"
                  username="resumo_dev" password="resumo_dev"
        />

troca o /beta pelo teu tenant