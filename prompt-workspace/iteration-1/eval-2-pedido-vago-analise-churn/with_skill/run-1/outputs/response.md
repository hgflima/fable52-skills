```text
Analyze our customer churn data and write a report that shows where and when we lose customers, and what we should do about it.

The data is at [FILL IN: the folder and file names, or the database and tables]. Our business is [FILL IN: one or two sentences on what you sell, how customers pay, and what counts as a churned customer, for example "a monthly subscription that was cancelled or not renewed"].

I will use this analysis to decide where to focus retention work over the next few months. The report will be read by the people who run product and customer success. They know the business well, they are not statisticians, and they have not seen the data. They need to come away knowing which groups of customers leave most, at what point in their life as a customer, and how sure we can be about each of those statements.

Get to know the data before you measure anything: what each file or table holds, what one row represents, which period it covers, and how complete it is. Problems in the data change the answer, so look for them on purpose: duplicated customers, missing values, cancellation dates that come before sign-up dates, and customers who signed up too recently to have had the chance to churn. If the data does not mark churn in an obvious way, or my definition above does not match what is in the data, choose the most defensible definition, state it at the top of the report, and continue.

Then answer these questions, as far as the data allows:
- What is the churn rate overall, and how has it moved over the period? Compare cohorts by sign-up month where there is enough history.
- At what point in a customer's lifetime does most churn happen?
- In which groups is churn concentrated? Use whichever attributes the data has, such as plan, price, acquisition channel, region, company size, product usage, or support contacts.
- If the data has revenue, how much revenue does churn cost, and does the picture change when you weight by revenue and not by number of customers?
- What does the data not let us answer, and which additional data would close that gap?

Hold each finding to a standard that a sceptical reader would accept. Give each number together with the count of customers behind it, and say so when a group is too small for a difference to mean much. This data shows which customers leave, not why they leave, so describe a pattern as an association unless something in the data supports a cause, and check whether an apparent effect is explained by another attribute, such as tenure or plan. I would rather read "the data cannot tell us" than a confident guess.

The goal is to describe and explain churn, so a model that predicts which customers will churn is out of scope. If you conclude that one would be worth building, say so in the next steps.

Treat the source data as read-only, because other people and systems depend on it: do not edit, move, or delete the original files, and if the data is in a database, run only read queries. Put everything you create in a new folder named `churn-analysis/`. Reading the data, running scripts, and installing packages in a local environment need no approval. The data may hold personal information, so keep every output at the aggregate level, with no names, emails, or other fields that identify a customer, and do not send the data to any external service.

The work is done when:
- `churn-analysis/report.md` exists and answers the questions above.
- `churn-analysis/` holds the script or notebook that produces every number and chart in the report from the source data, so that someone else can run it again.
- The report states the churn definition, the period covered, the number of customers analyzed, and any rows you excluded and why.

Write the report in Brazilian Portuguese and keep it to about three pages. Open with the three to five most important findings, each in one or two sentences with its key number, ordered by how many lost customers (or how much lost revenue) each one accounts for. Follow with a short section for each finding that shows the evidence, with a table or a chart where it makes the point faster than text, and save the charts as PNG files in the same folder. Then give up to three recommended actions, each tied to the finding that supports it, and close with the limits of the analysis and the open questions. Use plain words and full sentences, define each metric the first time it appears, and do not use a label you made up during the work without saying what it is.

Before you finish, run the script from start to end one more time and check every number in the report against its output. Close your final message with the main findings in a few sentences and the paths of the files you created.
```

Salvo em `prompts/churn-data-analysis.md`. Antes de enviar, preencha os dois trechos `[FILL IN: ...]` do segundo parágrafo: onde estão os dados, e qual é o negócio e o que conta como churn.

**Effort**: `high`. É uma análise exploratória com verificação dos números, e esse é o nível padrão quando há dúvida.

**Suposições**

O pedido deixava três pontos em aberto que mudam o prompt. Eu faria estas perguntas, e segui com a opção marcada:

1. Que decisão a análise apoia? Supus **entender onde e quando vocês perdem clientes, para priorizar retenção**. As alternativas eram prever quais clientes vão cancelar, só medir a evolução da taxa, ou avaliar uma ação já feita.
2. Onde estão os dados? Supus **arquivos em uma pasta, com o modelo rodando no Claude Code**. O prompt também cobre banco de dados (só consultas de leitura). Se você for colar os dados no chat, o prompt precisa mudar.
3. Quem lê o resultado? Supus **liderança de produto e de customer success, com um relatório de cerca de três páginas**. As alternativas eram um notebook para um time de dados ou um resumo de uma página para a diretoria.

Outras suposições:

- O relatório sai em português do Brasil, porque você escreveu em português. O prompt fica em inglês.
- Um modelo preditivo de churn fica fora do escopo. O prompt pede só que o modelo diga se vale a pena construir um.
- Os dados de origem são somente leitura, e tudo o que o modelo criar vai para `churn-analysis/`.
- Os dados podem ter informação pessoal de clientes, então o relatório fica no nível agregado e nada é enviado para serviços externos.
- Alguém acompanha a execução, ou ela é curta o bastante para não precisar de retomada.

**Adicionado para o Fable 5.2**

- Limites com ações nomeadas: o que é proibido (alterar os dados de origem, enviar dados para fora) e o que não precisa de aprovação (ler, rodar scripts, instalar pacotes localmente). Sem isso, o modelo tende a segurar trabalho de rotina. Sem custo.
- Uma frase para o caso de a definição de churn ser ambígua: o modelo escolhe, declara no topo do relatório e continua, em vez de parar para perguntar.
- Não incluí o bloco "Finishing work". A tarefa tem uma entrega só. Se você for deixar rodando sozinho sobre uma base grande, vale acrescentar. A Anthropic mediu cerca de 30% a mais de tokens de saída com ele.
- Não incluí atualizações de progresso nem o bloco completo "Communicating clearly". O formato do relatório já está descrito direto no prompt.
- O prompt pede o número e a contagem de clientes por trás de cada achado. Ele não pede que o modelo mostre o raciocínio, que o Fable 5.2 pode recusar.
