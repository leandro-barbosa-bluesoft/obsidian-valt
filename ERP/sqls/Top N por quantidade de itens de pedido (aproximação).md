SELECT pm.pedido_key,  
       fd.fornecedor_key,  
       COUNT(*) AS qtd_itens_pedido  
FROM pedido_master pm  
         JOIN fornecedor_divisao fd ON fd.fornecedor_divisao_key = pm.fornecedor_divisao_key  
         JOIN pedido p              ON p.pedido_key = pm.pedido_key  
         JOIN item_pedido ip        ON ip.pelt_key   = p.pelt_key  
GROUP BY pm.pedido_key, fd.fornecedor_key  
ORDER BY qtd_itens_pedido DESC  
    FETCH FIRST 50 ROWS ONLY;