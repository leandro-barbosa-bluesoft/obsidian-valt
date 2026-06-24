
SQL para consultar um pedido específico:

SELECT                                                                                
         pm.pedido_key,                                                                    
         pm.usuario_key AS comprador_key,                                                  
         u.user_name AS nome_consulta_pedidos,                                             
         p.nome_abreviado AS nome_linha_compra,                                            
         p.nome_razao AS nome_razao_comprador,                                             
         fd.fornecedor_key,                                                                
         fd.divisao_key,                                                                   
         fd.descricao AS fornecedor_divisao                                                
     FROM pedido_master pm                                                                 
     JOIN usuario u ON u.usuario_key = pm.usuario_key                                      
     JOIN pessoa p ON p.pessoa_key = u.usuario_key                                         
     JOIN comprador c ON c.pessoa_key = p.pessoa_key                                       
     JOIN fornecedor_divisao fd ON fd.fornecedor_divisao_key = pm.fornecedor_divisao_key   
     WHERE pm.pedido_key = :pedidoKey;

SQL para gerar o de-para geral entre login da Consulta de Pedidos e nome da Linha de Compra:

SELECT                                                                                
         c.pessoa_key AS comprador_key,                                                    
         u.user_name AS nome_consulta_pedidos,                                             
         p.nome_abreviado AS nome_abreviado_pessoa,                                        
         p.nome_razao AS nome_linha_compra,                                                
         p.status AS status_pessoa                                                         
     FROM comprador c                                                                      
     JOIN pessoa p ON p.pessoa_key = c.pessoa_key                                          
     LEFT JOIN usuario u ON u.usuario_key = c.pessoa_key                                   
     ORDER BY p.nome_razao;

SQL para ver os produtos/linhas de compra de um comprador e fornecedor:

SELECT                                                                                
         lc.comprador_key,                                                                 
         p.nome_razao AS comprador_linha_compra,                                           
         lc.fornecedor_key,                                                                
         lc.divisao_key,                                                                   
         fd.descricao AS fornecedor_divisao,                                               
         lc.produto_key,                                                                   
         pr.descricao AS produto,                                                          
         lc.loja_key                                                                       
     FROM linha_compra lc                                                                  
     JOIN pessoa p ON p.pessoa_key = lc.comprador_key                                      
     JOIN fornecedor_divisao fd                                                            
         ON fd.fornecedor_key = lc.fornecedor_key                                          
        AND fd.divisao_key = lc.divisao_key                                                
     JOIN produto pr ON pr.produto_key = lc.produto_key                                    
     WHERE lc.comprador_key = :compradorKey                                                         
       AND lc.fornecedor_key = :fornecedorKey                                                       
       AND lc.divisao_key = :divisaoKey                                                              
     ORDER BY pr.descricao, lc.loja_key;