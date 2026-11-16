```text
Fix the flaky tests in the checkout service: the tests that pass on some runs and fail on others with no change to the code.

The goal is a test suite whose failures can be trusted, so that a failing test means something is broken. That goal rules out the shortcuts. Do not make a test pass by deleting it, skipping it, loosening what it asserts, wrapping it in automatic retries, or lengthening a sleep or timeout until the failure becomes rare. Each of those hides the cause and leaves it in place.

Start by finding out how this repository runs the checkout service's tests (the README, the package scripts, or the CI configuration will say) and which tests are flaky. Use the CI history if you can reach it, and run the suite repeatedly to see the failures for yourself. For each flaky test, reproduce the failure and find its cause before you change anything.

Fix the cause where it is. If it is in the test, fix the test. If it is in the service code, for example a real race condition that the test exposes only sometimes, fix it there, keep that change narrow, and put it first in your recap, because a change to checkout behaviour needs a closer review than a change to a test.

Treat this as code that other people maintain and will review. Keep the diff as small as the fixes allow. If you find other problems nearby, list them at the end and leave them as they are.

You may edit files and run tests as often as you need without asking. Ask for my approval before you delete any file, and tell me which file and why. The exception is temporary files that you created during this task: remove those when you are done.

The work is done when:
- Every test you identified as flaky passes on many consecutive runs. Choose the number of runs from the failure rate you measured before the fix, so that the old failure would almost certainly have appeared. For example, a test that failed about 1 run in 20 needs about 100 clean runs.
- The full test suite of the checkout service passes.
- No test was deleted, skipped, or weakened.

If you cannot reproduce a failure, or cannot find the cause of one, say so and leave that test as it is. Do not guess at a fix.

Before you start, say in a line what you're about to do; brief updates while you work help the user follow along. Close with a short recap that stands on its own (what you found, what you did, and what's next), so a reader who only sees the last message has the full picture.

In the recap, give for each flaky test: its name, the cause and the evidence that showed it, the change you made, and how many runs passed in a row afterwards. Then list the files you changed, and the problems you noticed and left alone.
```

Salvo em `prompts/fix-flaky-checkout-tests.md`.

O comportamento estranho tem uma causa provável: o prompt antigo pedia duas coisas opostas ("mostre todo o raciocínio" e "responda sem deliberar"), e o Fable 5.2 segue cada linha ao pé da letra. Mande o prompt novo em uma conversa nova, para as instruções antigas não ficarem no histórico.

**Effort:** `high`. Achar a causa de uma falha intermitente é depuração difícil e exige muitas execuções de teste.

**Suposições**

Eu faria três perguntas antes de escrever. Sem resposta, assumi a opção marcada:

1. O "NEVER delete files" é proibição total ou você quer aprovar antes? Assumi **aprovar antes**. Se for proibição total, troque a frase por "Do not delete any file" e diga o motivo.
2. Menor diff possível ou o modelo pode reestruturar o código? Assumi **menor diff**, com os problemas vizinhos listados e não corrigidos.
3. Você acompanha a execução ou deixa rodando sozinho? Assumi **que você acompanha** e responde aos pedidos de aprovação.

Outras suposições:

- Não sei quais testes são flaky nem qual é o comando de teste, então o prompt manda o modelo descobrir. Se você souber, coloque os nomes dos testes e o comando no primeiro parágrafo. O prompt fica bem melhor.
- Defini "pronto" como muitas execuções seguidas sem falha, com o número calculado pela taxa de falha medida antes. O seu prompt não dizia como conferir.
- Proibi os atalhos (apagar, pular, afrouxar o teste, retry automático, aumentar sleep), porque eles escondem a causa.
- Se a causa estiver no código do serviço e não no teste, o modelo corrige e destaca isso no início do resumo. Se você preferir aprovar antes, peça isso no prompt.

**Adicionado para o Fable 5.2**

- O texto de progresso (o parágrafo "Before you start…"), porque assumi que você acompanha. Ele cobre o que você queria com "mostre o raciocínio": saber o que o modelo está fazendo. A Anthropic mediu cerca de 20% a mais de tokens de saída com ele, em system prompts. Aqui o tamanho do efeito não foi medido.
- A postura de diff mínimo, sem a frase de "dono do código". Sem custo.
- Não incluí o bloco "Finishing work". Se você for deixar rodando sozinho ou usar effort `medium`, me avise e eu adiciono (cerca de 30% a mais de tokens de saída).

**Removido**

- "Think step by step and show your full reasoning in `<thinking>` tags": o raciocínio já está sempre ligado, e o pedido de reproduzi-lo na resposta pode ser recusado pelo modelo. No lugar, o resumo pede a evidência de cada causa.
- "Answer directly without deliberating": contradiz a linha anterior e não reduz o raciocínio em effort `medium` ou maior. Para respostas mais rápidas, baixe o effort.
- "You have a budget of 200,000 tokens": um orçamento no texto muda quanto o modelo gasta. A Anthropic viu custo menor e resultado equivalente sem ele.
- "Use subagents for everything you can": aumenta a delegação e o custo. O modelo decide sozinho quando delegar.
- "Keep all sentences under 15 words": o modelo obedece à risca e o texto fica picotado. No lugar, o prompt descreve o que o resumo deve conter.
- "CRITICAL: NEVER delete files": o modelo trata "never" como absoluto, o que proíbe até apagar os arquivos temporários que ele mesmo criou. Virou pedido de aprovação, sem maiúsculas.
- "You are an expert engineer": o prompt agora abre com a tarefa. Se você quiser a linha de papel, o lugar dela é o system prompt.

Quando o modelo pedir aprovação para apagar algo, responda de forma específica ("Sim, pode apagar `tests/fixtures/old_cart.json`"). Funciona melhor que "continua".
