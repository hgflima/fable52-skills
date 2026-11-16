Preciso que você analise os dados de churn de [EMPRESA/PRODUTO] e me diga onde estamos perdendo clientes, por quê (até onde os dados permitem afirmar) e onde vale a pena agir primeiro.

## Contexto do negócio

[2 a 4 frases: o que a empresa vende, como cobra (assinatura mensal, anual, contrato, uso), quem é o cliente (B2B, B2C, porte) e por que o churn virou assunto agora.]

Esta análise vai servir para [DECISÃO, por exemplo: escolher onde o time de retenção concentra esforço no próximo trimestre]. Quem vai ler é [PÚBLICO, por exemplo: diretoria de produto e de CS, sem formação em estatística]. O que importa para esse leitor é o que os números significam para a decisão, e não a técnica usada para chegar neles.

## Os dados

[Onde estão: arquivos anexados, caminho no disco ou tabelas do banco. Para cada fonte: o que cada linha representa, período coberto, colunas principais e qualquer problema que você já conheça.]

Definição de churn a usar: [por exemplo: cliente pagante que cancelou ou não renovou; ou cliente sem atividade há 60 dias].

Se essa definição não puder ser aplicada aos dados como estão (por exemplo, não existe data de cancelamento), use a definição mais próxima que os dados permitem, explique a diferença e siga com ela.

## O que quero saber

Em ordem de importância:

1. Tamanho e tendência. Qual é a taxa de churn, como ela evoluiu no período, e a variação é real ou é efeito de mudança no mix de clientes? Onde os dados permitirem, separe churn de clientes de churn de receita, porque perder muitos clientes pequenos e perder poucos clientes grandes pedem respostas diferentes.
2. Quando. Em que ponto do ciclo de vida os clientes saem: nos primeiros meses, na primeira renovação, depois de um reajuste de preço?
3. Quem. Quais segmentos (plano, canal de aquisição, porte, região ou outros que os dados tragam) têm churn claramente acima ou abaixo da média.
4. Sinais anteriores. Que comportamentos aparecem antes do cancelamento (queda de uso, chamados de suporte, falha de pagamento) e com quanta antecedência.
5. Onde agir. Juntando tudo, onde está a maior perda que parece evitável e o que você testaria primeiro.

Se os dados não permitirem responder alguma dessas perguntas, diga isso e diga qual dado faltaria. Prefiro uma lacuna declarada a uma resposta apoiada em aproximação fraca.

## Como conduzir a análise

Rode a análise de verdade sobre os dados, com código. Todo número do relatório deve sair de um cálculo que você executou. Se você não conseguir acessar ou ler os dados, pare e me avise; uma análise ilustrativa com números plausíveis não me serve e pode ser confundida com resultado real.

Antes de calcular qualquer taxa, inspecione os dados: período realmente coberto, valores ausentes, duplicatas, clientes sem data de início, datas impossíveis, chaves que não batem entre tabelas. Relate o que encontrou e como tratou, porque isso determina quanto dá para confiar no resto.

Análises de churn costumam errar nos mesmos lugares. Peço atenção a estes:

- Clientes recentes ainda não tiveram tempo de cancelar. Comparar a taxa de churn de quem entrou há dois meses com a de quem entrou há dois anos faz as safras novas parecerem melhores do que são. Compare safras no mesmo tempo de vida (retenção por coorte) ou use análise de sobrevivência.
- Algumas variáveis só existem por causa do cancelamento, ou são medidas depois dele: motivo de cancelamento, uso zerado no último mês, contato com o time de retenção. Elas descrevem a saída, mas não servem como sinal antecipado. Para a pergunta 4, use apenas o que era observável antes da data de churn.
- Os dados são observacionais. Um segmento com churn alto pode diferir dos outros em várias coisas ao mesmo tempo. Descreva as relações como associações, aponte os fatores de confusão mais prováveis e, quando uma conclusão causal for importante para a decisão, diga qual experimento a confirmaria.
- Segmentos pequenos produzem taxas extremas por acaso. Mostre sempre o número de clientes ao lado de cada taxa e não dê destaque a diferenças que se apoiam em poucos casos.
- Churn voluntário (o cliente decidiu sair) e involuntário (cartão recusado, inadimplência) têm causas e soluções diferentes. Separe os dois se os dados permitirem.

Se houver uma ambiguidade grande o bastante para mudar as conclusões (por exemplo, duas formas plausíveis de contar churn que dão taxas muito diferentes), calcule das duas formas, mostre a diferença e diga qual você recomenda. Para dúvidas menores, escolha a interpretação mais razoável, registre a escolha e continue.

## O que entregar

Um relatório em português, escrito para o público descrito acima, nesta ordem:

1. Conclusões principais, de 3 a 5, cada uma com o número que a sustenta e o grau de confiança. Quem ler só esta parte deve conseguir tomar a decisão.
2. A análise de cada pergunta, com tabelas e gráficos onde eles ajudarem a enxergar o padrão.
3. Recomendações em ordem de prioridade, cada uma com o tamanho estimado da perda que ela ataca e como medir se funcionou.
4. Limitações e premissas: qualidade dos dados, definição de churn usada, o que você assumiu e o que não deu para responder.

Em todo o relatório, deixe claro o que é resultado calculado, o que é interpretação sua e o que é hipótese a testar. Entregue também o código usado, para que a análise possa ser reproduzida e refeita com dados novos.
