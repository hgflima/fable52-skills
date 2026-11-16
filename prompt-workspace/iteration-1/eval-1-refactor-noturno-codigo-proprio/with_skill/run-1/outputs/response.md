```text
Refactor the payments code in this Rails monolith so that tax calculation lives in one place. Today the same tax calculation logic is duplicated across four service objects in `app/services/billing/`.

The duplication costs us every time a tax rule changes: the change has to be made four times, and a copy that gets missed charges customers the wrong amount. After this work, each tax rule should have a single implementation that all four services call.

You are the owner and long-term maintainer of this codebase, and its design is up to you. Restructure whatever the consolidation needs: extract a new class or module, change the interfaces of the four services, and update their callers elsewhere in the app.

This code decides how much customers are charged, so the refactor must not change any amount that the current code produces. The order of the first steps matters for that reason. Before you change any production code, find the four services and any other copy of the tax logic, run the full test suite to record which tests already fail, and write characterization tests that pin what each service computes today. Cover the cases the code itself branches on, such as rounding, zero and negative amounts, exemptions, and different rates. Confirm that these tests pass against the unmodified code and commit them. They are how both of us will know that behaviour was preserved.

The four copies have probably drifted apart. Where two of them give different results for the same input, keep the current result of each service, make the difference explicit in the shared code (a named option, not a silent branch), and list it in the report. The same goes for anything that looks like a bug in the calculation: preserve it and report it. Which variant is correct is a tax decision for me to make, not something to settle inside a refactor.

I have not given you the test command. Use the one that CI runs. The CI configuration and the presence of `spec/` or `test/` will tell you which it is.

The work is done when:
- The tax calculation has one implementation, all four services call it, and no copy of it remains in `app/services/billing/` or anywhere else you found one.
- The characterization tests you wrote against the original code pass, unchanged, against the refactored code.
- The full test suite passes on the final commit, apart from the failures you recorded before you started.
- No test was deleted, skipped, or loosened to get a pass. If something cannot be made to pass, leave it failing and explain why in the report.

I will leave this running overnight and will not be available, so there is nobody to ask. Work on a new local branch named `billing-tax-consolidation`. Commit the characterization tests first, then the shared implementation, then one commit for each service you move onto it, so that I can review and revert service by service. Do not push the branch, open a pull request, or deploy. Those stay with me. Do not send requests to the real payment gateway or tax provider. Verify through the test suite and the stubs it already has. Setting up the local test database to run the suite is routine and fine. I do not expect this refactor to need a schema change. If you conclude that it does, do not write the migration, and describe it in the report. When you reach a decision that you would normally bring to me, take the option that changes behaviour least, carry on, and record the decision in the report.

# Finishing work
Please avoid stopping while you still owe work the user asked for. Be persistent and do not stop unless needed.

While status notes are welcome, as is clarifying with the user on open decisions, carry on with work that does not depend on the user's answer without unnecessary stops.

If you notice yourself inviting the user to redirect you or offering to wait, instead proceed on the next part of the task. You may meet ordinary obstacles like errors, timeouts, locked files, empty results or failing tools. Diagnose them first, and when they are not real blockers, work through them with the access you have (wait and retry, fix the request, use another tool or source) rather than stopping unnecessarily.

If someone clearly decided something is a hard blocker, such as a file marked not to be touched, access that was intentionally withheld, or a safety guardrail, leave it alone, say plainly what you found, and look for another safe way to finish the task. Stopping before the task is complete is very rarely merited, as in when the task can't move forward without user input, or where the blocker is deliberate and should not be worked past. This does not override the need for confirmation on risky or destructive actions.

When you finish, write a report as your final message that I can read in the morning without having watched the run. Start with whether the work is complete and whether the suite passes, with the exact command you ran and its result. Then say where tax calculation lives now and how each of the four services uses it. After that, list every difference you found between the original copies and how you kept it, every change you made beyond the consolidation (renamed or moved code, changed interfaces and their callers, new tests, anything else, small changes included, because I will check the diff against this list), the decisions you took in my place, and anything unfinished or failing with what it needs. Write in plain full sentences, say what any name you introduced refers to, and keep the report to about one page unless the lists need more.
```

Arquivo salvo em `/Users/henriquelima/dev/personal/fable52-prompts/fable52-prompt-workspace/iteration-1/refactor-noturno-codigo-proprio/with_skill/outputs/prompts/refactor-billing-tax-calculation.md` (contém só o prompt).

**Effort:** `high`. É um refactor difícil, em código de cobrança, rodando a noite inteira sem ninguém para dizer "continua".

**Suposições**

Eu não tinha como perguntar, então estas três perguntas ficaram respondidas por suposição. Se alguma estiver errada, me diga e eu ajusto o prompt:

1. Qual comando roda os testes (`bundle exec rspec`, `bin/rails test`, outro)? O prompt manda o modelo usar o que a CI roda. Se você souber o comando, troque esse parágrafo por ele.
2. Se as quatro cópias derem resultados diferentes para a mesma entrada, o que fazer? Assumi: preservar o resultado atual de cada service, deixar a diferença explícita no código e listar no relatório. Qual variante está certa é decisão fiscal sua.
3. Como você quer encontrar o trabalho de manhã? Assumi: commits em um branch local novo (`billing-tax-consolidation`), um por service, sem push e sem PR.

Outras suposições:

- Não havia repositório para consultar, então o prompt não cita os nomes dos quatro services. O modelo os encontra. Se você listar os quatro arquivos no primeiro parágrafo, ele começa mais rápido.
- "Pode reestruturar o que precisar" inclui mudar as interfaces dos services e os chamadores fora de `app/services/billing/`.
- O refactor não deve mudar nenhum valor cobrado. Por isso o prompt pede testes de caracterização antes de qualquer mudança.
- Não deve precisar de mudança de schema. Se precisar, o modelo descreve no relatório e não escreve a migration.
- Nada de chamadas ao gateway de pagamento ou ao provedor fiscal reais durante a noite.

**Adicionado para o Fable 5.2**

- **Frase de ownership** ("You are the owner and long-term maintainer…"), porque o código é de vocês e a reestruturação é bem-vinda. A Anthropic mediu consolidação de código duplicado em cerca de 75% das execuções com a frase, contra menos de 10% sem ela. O custo medido foi de 70 a 80% mais tokens de saída, e mudanças que ninguém pediu (testes e documentação novos, às vezes mudança de comportamento). Em pagamentos isso pesa: revise o diff completo de cada commit, não só o relatório. O prompt pede que o relatório liste também as mudanças pequenas, que o modelo costuma omitir.
- **Bloco "Finishing work"**, porque ninguém vai estar lá para destravar. A Anthropic mediu cerca de 30% mais tokens de saída. Com ele o modelo pergunta menos, então o prompt nomeia o que fica com você: push, PR, deploy e migrations.
- **Relatório final** pedido em um parágrafo com o conteúdo e o tamanho definidos, no lugar do bloco "Communicating clearly" completo. O leitor é uma pessoa só e o prompt já diz o que o relatório precisa ter.
- **Sem texto de progresso e sem menção a subagentes**, porque ninguém acompanha a execução e o trabalho é uma linha só.

Esses números são da Anthropic, medidos em system prompts nas tarefas dela. Em um prompt de tarefa a direção do efeito deve ser a mesma, mas o tamanho não foi medido.

Uma coisa fora do prompt: para rodar a noite toda, a sessão precisa ter permissão para editar arquivos, rodar os testes e fazer commits sem pedir confirmação. Se ela parar em um pedido de permissão, você encontra o trabalho parado de manhã.
