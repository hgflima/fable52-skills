# Perguntas para o usuário

1. Qual é o negócio e como vocês definem churn hoje? (Cancelamento de assinatura, não renovação de contrato, inatividade por N dias? Contam clientes, receita ou os dois?)
2. Que dados você tem e onde o prompt vai rodar? (Quais tabelas ou arquivos, que período cobrem, e se é um chat com upload de arquivo, um agente com acesso ao banco ou uma chamada de API dentro de um pipeline.)
3. Que decisão essa análise vai apoiar e quem vai ler o resultado? (Por exemplo: priorizar ações de retenção para a diretoria, ou montar um modelo de previsão para o time de dados.)

# Premissas adotadas na falta de resposta

- Negócio de receita recorrente (assinatura ou contrato), com churn definido como cancelamento ou não renovação. A definição ficou como campo a preencher no prompt.
- O modelo que recebe o prompt consegue executar código sobre os dados (chat com análise de arquivos ou agente com acesso a arquivos/banco).
- Objetivo: análise diagnóstica para orientar ações de retenção, e não a construção de um modelo preditivo em produção.
- Leitores sem formação em estatística; relatório em português.
- Uso pontual do prompt, e não uma rotina recorrente automatizada.
