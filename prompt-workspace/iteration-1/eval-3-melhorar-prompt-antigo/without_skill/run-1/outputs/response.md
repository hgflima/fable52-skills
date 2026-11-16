O comportamento estranho vem principalmente de o prompt se contradizer e de a tarefa em si ocupar uma única frase no meio de regras de estilo. Modelos recentes seguem instruções de forma mais literal, então cada regra exagerada ou conflitante acaba sendo obedecida, em vez de ignorada como o modelo antigo fazia.

## O que estava causando problema

| Trecho original | Problema | O que fiz |
|---|---|---|
| "Think step by step and show your full reasoning in `<thinking>` tags" + "Answer directly without deliberating" | As duas instruções se contradizem, e o modelo oscila entre elas. | Removi as duas. O modelo já raciocina sozinho; se você quiser ver o raciocínio, ative o thinking na API ou no harness, não no texto do prompt. |
| "CRITICAL: NEVER delete files" | Caixa alta e "NEVER" sem motivo fazem o modelo aplicar a regra com rigidez excessiva, por exemplo recusando limpar um temporário que ele mesmo criou. | Reescrevi em tom normal, com escopo, exceção e o que fazer quando apagar parecer a correção certa. |
| "You have a budget of 200,000 tokens" | O modelo não mede bem o próprio consumo, e o aviso tende a fazê-lo apressar ou encerrar cedo. | Removi. Troquei por uma orientação sobre o que fazer se o trabalho for maior que o esperado. |
| "Use subagents for everything you can" | Seguido ao pé da letra, o modelo delega até tarefas triviais, o que custa caro e perde contexto. | Limitei a trabalho independente e paralelizável. |
| "Keep all sentences under 15 words" | Vale para tudo, inclusive explicações técnicas e comentários de código, e deixa o texto truncado. | Troquei por um formato de relatório final curto. |
| "You are an expert engineer" | Não acrescenta informação. | Removi. |
| "Fix the flaky tests in the checkout service" | Era a única frase sobre a tarefa, sem critério de pronto. O risco é o modelo "resolver" com retry, skip ou sleep. | Virou o centro do prompt, com objetivo, critério de correção e verificação. |

## Prompt reescrito

Mantive em inglês, como o original.

```
Fix the flaky tests in the checkout service.

## Context

Some tests in the checkout service pass and fail intermittently with no code change, which makes CI results untrustworthy. The goal is a suite that passes reliably because the source of nondeterminism is gone, not one where the flakiness is hidden.

[If known, list here: which tests flake, how often, whether it happens only in CI or also locally, and the command that runs the suite.]

## What a good fix looks like

Reproduce each flaky test before changing it: run it repeatedly, in isolation and together with the rest of the suite, in random order if the runner supports it. A fix for a failure you never observed is a guess.

Fix the cause of the nondeterminism: shared state between tests, order dependence, real clocks or timers, unmocked network calls, random data, race conditions in async code, and so on. Retries, skips, longer sleeps or timeouts, and loosened assertions hide the problem instead of fixing it, so avoid them unless one is genuinely the correct fix, and say so in the report when it is.

If a test is flaky because the checkout code itself has a real bug (for example a race condition), fix the code when the change is small and contained. If it needs a larger change, leave it and describe it in the report.

Verify before reporting: run each fixed test at least 20 times in a row, then run the full checkout test suite once.

## Constraints

Do not delete existing files in the repository, including test files, fixtures and snapshots. [State the reason here, e.g. "other teams depend on these fixtures".] If you conclude that deleting a file is the right fix, leave the file in place and recommend the deletion in the report. Cleaning up temporary files you created yourself is fine.

Keep changes inside the checkout service and its tests. If the cause lives elsewhere (shared test utilities, CI configuration, another service), report it instead of changing it.

## Working style

Do small or sequential work yourself. Use subagents only where the work is independent and can run in parallel, such as investigating several unrelated flaky tests at once.

Take the time the investigation needs. If the job turns out to be much bigger than expected, fix what you can fix properly and report the rest, rather than applying shallow fixes everywhere.

## Final report

When you finish, write a short report in plain prose covering:

- for each flaky test: the root cause, what you changed, and how you verified it (including how many runs passed)
- anything you could not fix or chose not to fix, and why
- the list of files you changed
```

## Antes de rodar

Há dois trechos entre colchetes para você preencher ou apagar:

- **Contexto dos testes:** quais testes falham, com que frequência e o comando da suíte. Sem isso o prompt funciona, mas o modelo gasta tempo descobrindo.
- **Motivo da regra de não apagar arquivos:** com o motivo, o modelo acerta melhor os casos de borda.

Assumi algumas coisas que vale você confirmar:

- **Código de produção:** o agente pode corrigi-lo quando a mudança for pequena. Se ele só puder mexer nos testes, troque esse parágrafo.
- **Limite de tokens:** tratei como não sendo uma restrição real. Se for, imponha o limite no harness, que é onde ele é de fato controlado.
- **Regra das 15 palavras:** entendi que o objetivo era resposta curta. Se era outra coisa, me diga qual.

Não testei o prompt em execução; a reescrita se baseia nos problemas visíveis no texto original.
