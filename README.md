# fable52

Plugin do Claude Code com uma skill, `prompt`, que transforma um pedido solto em um prompt de tarefa
pronto para colar no Claude Fable 5.2 (e no Claude Mythos 5.2), seguindo o guia de prompting da
Anthropic para esse modelo.

O Fable 5.2 segue instruções de forma literal. Um prompt escrito para um modelo anterior costuma
trazer linhas que ele obedece demais (um "never" solto, um limite de palavras por frase, uma ordem
para delegar tudo) e costuma não dizer o que ele precisa saber (de quem é o código, se alguém
acompanha a execução, o que significa "pronto"). A skill corrige as duas coisas.

## O que você recebe

- O prompt em inglês, em um bloco pronto para copiar.
- O arquivo `prompts/<slug>.md`, que contém só o prompt. Um arquivo existente não é sobrescrito: o
  novo ganha um sufixo numérico.
- O nível de effort recomendado, com o motivo.
- As suposições feitas, uma por linha.
- Os textos acrescentados para o Fable 5.2 e o custo de cada um.
- Ao melhorar um prompt existente: o que foi removido e por quê.

As notas vêm no idioma em que você escreveu. O prompt sai sempre em inglês.

Dois limites:

- A skill escreve o prompt e não executa a tarefa que ele descreve.
- Ela escreve prompts de tarefa (mensagens de usuário), não system prompts.

## Instalar

```bash
claude plugin marketplace add /caminho/para/fable52-skills
claude plugin install fable52@hgflima-fable52
```

Para testar sem instalar:

```bash
claude --plugin-dir /caminho/para/fable52-skills
```

## Usar

A skill dispara sozinha com pedidos como "cria um prompt para…", "melhora esse prompt" ou
"como eu peço isso pro Fable". Para chamar direto:

```
/fable52:prompt arrumar o bug do 401 depois que o token expira
```

## Como a skill trabalha

1. **Entende o pedido.** Se a tarefa é sobre o repositório atual, ela consulta os nomes reais
   (caminhos, comando de teste, gerenciador de pacotes) para o prompt dizer `pnpm -r test` e não
   "rode os testes".
2. **Pergunta só o necessário.** No máximo três perguntas de múltipla escolha, em uma única rodada,
   e só quando um palpite errado mudaria o prompt. O que continuar em aberto vira suposição
   declarada.
3. **Escreve o núcleo.** A tarefa como instrução, o motivo, um critério de pronto que alguém
   consegue verificar, os limites com seus motivos, e o leitor e o formato do resultado.
4. **Acrescenta os textos da Anthropic** só quando a condição vale (tabela abaixo).
5. **Remove o que atrapalha o Fable 5.2** (lista abaixo).
6. **Recomenda um nível de effort** e entrega.

### Textos acrescentados

| Quando a tarefa… | A skill acrescenta | Custo |
|---|---|---|
| é longa e roda sem ninguém olhando | o texto "Finishing work" | cerca de 30% mais tokens de saída |
| altera código e você quer a estrutura melhorada | a frase de ownership | cerca de 70 a 80% mais tokens de saída, e mudanças que ninguém pediu |
| altera código e você quer a menor mudança | um pedido de diff mínimo, sem a frase de ownership | nenhum |
| tem alguém acompanhando a execução | o texto de progresso | cerca de 20% mais tokens de saída |
| inclui ações destrutivas ou visíveis para outras pessoas | as ações exatas que precisam de aprovação | nenhum |

Os números são da Anthropic e foram medidos em system prompts. A skill coloca os textos em um
prompt de tarefa, onde o tamanho do efeito não foi medido.

### Linhas removidas

- Pedidos para mostrar o raciocínio ("think step by step", tags `<thinking>`).
- Pedidos para pensar menos ("answer directly").
- Orçamento de tokens mostrado ao modelo.
- Ordens gerais para delegar a subagentes.
- Limites rígidos de estilo ("frases com menos de 15 palavras").
- Ênfase em maiúsculas ("CRITICAL", "YOU MUST").
- Cautela genérica ("be very careful"), linhas de prefill e papéis genéricos ("You are an expert
  engineer").

Um "never" vira "peça aprovação antes de X", a não ser que você queira uma proibição absoluta.

### Effort

O effort é uma configuração da requisição, não um texto do prompt.

| Nível | Para tarefas |
|---|---|
| `low` | rápidas, bem especificadas ou mecânicas |
| `medium` | de código interativo, chat ou trabalho de conhecimento do dia a dia |
| `high` | de código difícil, pesquisa, trabalho agêntico ou execução longa sem supervisão. É o padrão. |
| `xhigh` | entre as mais difíceis de código ou trabalho agêntico, quando o custo extra é aceitável |
| `max` | abertas, como matemática de pesquisa, quando tempo e tokens não importam |

## Resultados dos evals

Rodada 1, em 2026-11-16: três casos, uma execução por configuração.

| Caso | Com a skill | Sem a skill |
|---|---|---|
| 1. Refactor noturno em código próprio | 11 de 11 | 6 de 11 |
| 2. Pedido vago de análise de churn | 10 de 10 | 8 de 10 |
| 3. Melhorar um prompt antigo | 12 de 12 | 11 de 12 |
| **Total** | **33 de 33 (100%)** | **25 de 33 (76%)** |
| Tempo médio por execução | 114 s | 85 s |
| Tokens médios por execução | 79 mil | 57 mil |

Como ler esses números:

- Houve uma execução por configuração, então não há medida de variância. Diferenças pequenas podem
  ser ruído.
- As verificações medem o que a skill promete (inglês, effort, textos do guia do Fable 5.2). Elas
  não medem a qualidade geral do prompt. Os prompts sem a skill também são bons, e vale ler os dois
  lado a lado em `prompt-workspace/iteration-1/`.
- As 8 falhas sem a skill: prompt em português (2 de 3 casos), nenhuma recomendação de effort
  (3 de 3), e no caso 1 a falta da frase de ownership, do texto "Finishing work" e do aviso de custo.
- A skill custa cerca de 38% mais tokens por execução.

## Estrutura

```
.claude-plugin/
  plugin.json                 manifesto do plugin (fable52, versão 0.1.0)
  marketplace.json            marketplace de um plugin só (hgflima-fable52), apontando para a raiz
  icon.png                    ícone do plugin (1024 × 1024)
skills/prompt/
  SKILL.md                    a skill: fluxo, regras de escrita, effort e formato de entrega
  references/
    fable-5-2-patterns.md     textos da Anthropic para o Fable 5.2, quando usar cada um e o custo
    examples.md               três exemplos longos: código noturno, pesquisa e prompt a melhorar
  evals/evals.json            três casos de teste no formato do skill-creator
prompt-workspace/
  iteration-1/                saídas, notas e benchmark da rodada 1, com e sem a skill
```

A pasta `evals/` na raiz fica livre para uma suíte do `claude plugin eval`, que usa outro formato.

## Fontes

Os textos em `references/fable-5-2-patterns.md` vêm da documentação da Anthropic, lida em
2026-11-16. O arquivo lista os links: o guia de prompting do Fable 5.2, as boas práticas de
prompting, as novidades do modelo, o guia de migração e a página sobre effort.

## Licença

MIT. Veja o arquivo `LICENSE`.
