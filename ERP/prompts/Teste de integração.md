Caso precise descobrir quais dados pode utilizar da massa de dados do nosso banco de testes, pare a implementação, crie um sql de consulta e me peça para executar. 

A estrutura das tabelas estão definidas em erp-main-parent/bee/tables e outra estruturas menos importantes para nosso teste como sequences, triggers, views e afins são encontradas em erp-main-parent/bee

Utilize o guideline erp-integration-tests/.junie/erp-integration-tests-guidelines.md

Nunca execute todo o módulo de testes (exemplo: Run test /home/leandro.barbosa/dev/erp/erp-integration-tests) pois isso demora horas para finalizar, ao invés disso, execute APENAS a classe de testes que você está criando ou alterando.