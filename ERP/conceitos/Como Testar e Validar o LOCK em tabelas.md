Para verificar se a tabela `resumo_pedidos_pendentes_total` está bloqueada ou para analisar o estado da procedure durante a execução, utilize os seguintes comandos:

**1. Consulta Simples na Tabela (Teste de Bloqueio)** _Se este comando ficar "executando" por muito tempo, a tabela está com um `LOCK` que impede a leitura._

```
SELECT * FROM public.resumo_pedidos_pendentes_total LIMIT 1;
```

**2. Consulta de Estado da Procedure** _Verifica o que a procedure está fazendo: `active` (trabalhando) ou `idle` (ociosa), e se está esperando por algum evento (`wait_event`)._

```
SELECT
    pid,
    state,
    wait_event_type,
    wait_event,
    query
FROM pg_stat_activity
WHERE query ILIKE '%relatorio_pedidos_pendentes_tot%';
```

**3. Consulta do Processo Bloqueador** _Se a consulta acima mostrar um `wait_event_type` de `Lock`, este comando identificará exatamente qual processo (PID) e qual query estão causando o bloqueio._

```
SELECT
    activity.pid AS processo_bloqueado_pid,
    activity.query AS processo_bloqueado_query,
    blocking.pid AS processo_bloqueador_pid,
    blocking.query AS processo_bloqueador_query,
    age(now(), blocking.query_start) AS duracao_bloqueador
FROM pg_stat_activity AS activity
JOIN pg_locks AS locks ON locks.pid = activity.pid AND NOT locks.granted
JOIN pg_locks AS blocking_locks ON blocking_locks.locktype = locks.locktype
     AND blocking_locks.DATABASE IS NOT DISTINCT FROM locks.DATABASE
     AND blocking_locks.relation IS NOT DISTINCT FROM locks.relation
     AND blocking_locks.pid != locks.pid
JOIN pg_stat_activity AS blocking ON blocking.pid = blocking_locks.pid
WHERE locks.granted = false;
```