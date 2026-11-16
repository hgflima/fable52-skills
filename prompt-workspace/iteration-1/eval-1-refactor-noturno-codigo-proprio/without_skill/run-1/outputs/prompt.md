Quero que você refatore o módulo de pagamentos do nosso monolito Rails para eliminar a lógica duplicada de cálculo de imposto. Você vai trabalhar durante a noite, sem ninguém acompanhando. Eu leio o resultado amanhã de manhã.

## Contexto

O cálculo de imposto hoje está espalhado em quatro services dentro de `app/services/billing/`. Cada um carrega a própria cópia, ou variação, da mesma lógica. Qualquer mudança de regra fiscal precisa ser feita em quatro lugares, e já não temos certeza de que as quatro cópias ainda calculam a mesma coisa. O objetivo é ter uma única fonte de verdade para o cálculo de imposto, usada pelos quatro services.

O código é nosso e você tem liberdade para reestruturar o que for preciso: criar classes ou módulos, mover e renomear arquivos, mudar assinaturas internas, ajustar chamadores e testes. Não precisa preservar a estrutura atual só por ela existir. Escolha o desenho que você defenderia num code review, seguindo as convenções que já existem no repositório.

## O que não pode mudar

Esse código decide quanto cobramos de cada cliente, então o comportamento observável tem que continuar idêntico: para as mesmas entradas, cada um dos quatro fluxos precisa produzir os mesmos valores de hoje, no centavo e com o mesmo arredondamento.

O risco principal é que as quatro cópias provavelmente não são idênticas. Podem diferir em arredondamento, ordem das operações, base de cálculo, alíquotas ou tratamento de casos de borda (valor zero, desconto, isenção, estorno). Algumas diferenças podem ser intencionais e outras podem ser bugs antigos, e só lendo o código não dá para saber qual é qual. Por isso:

- Preserve o comportamento atual de cada fluxo, mesmo quando parecer errado.
- Onde os fluxos divergem, deixe a diferença explícita no novo desenho (um parâmetro, uma estratégia, o que fizer sentido), em vez de escolher uma versão e aplicá-la a todos.
- Liste cada divergência no relatório. Eu decido de manhã o que é bug e o que é regra.

## Como abordar

1. Comece lendo: os quatro services, quem os chama, os testes existentes. Procure também cálculo de imposto fora de `app/services/billing/` (models, jobs, helpers, rake tasks). Se achar, registre no relatório. Só mexa fora de `billing/` no que for necessário para a extração funcionar.
2. Antes de alterar qualquer coisa, rode a suíte e guarde o resultado como linha de base. Se já houver testes falhando, anote quais, para não confundi-los com regressões suas.
3. Antes de refatorar, escreva testes de caracterização que fixem o comportamento atual de cada um dos quatro services, incluindo os casos de borda que você encontrar na leitura. Eles têm que passar contra o código atual. São a sua rede de segurança e ficam no repositório.
4. Só então faça a extração, migrando um service de cada vez e rodando os testes a cada passo.
5. No fim, rode os testes de caracterização, a suíte completa e o linter do projeto, e compare com a linha de base.

## Trabalhando sem supervisão

Não há ninguém para responder perguntas. Quando houver dúvida, faça a escolha razoável mais conservadora, registre a decisão e o motivo, e siga em frente. Não pare esperando resposta.

Como ninguém está olhando para desfazer um erro, respeite estes limites:

- Trabalhe numa branch nova, criada a partir da branch atual. Faça commits pequenos e com mensagens claras, cada um deixando a suíte verde. Não faça push, não abra PR e não faça merge. Eu reviso antes de qualquer coisa sair da máquina.
- Use só o ambiente de desenvolvimento e teste. Nada de produção ou staging, nada de chamadas reais a gateway de pagamento ou serviço fiscal, nada de deploy.
- Não altere o schema, não crie migrations e não mexa em dados. Se a refatoração parecer pedir isso, descreva a proposta no relatório.
- Não adicione nem atualize gems. Se achar que uma ajudaria, proponha no relatório.
- Não enfraqueça testes para fazê-los passar: não apague, não marque como skip, não afrouxe asserções. Um teste existente que falha depois da refatoração é sinal de que o comportamento mudou. A exceção são testes acoplados à estrutura interna (por exemplo, um que testa um método privado que mudou de lugar): esses podem ser movidos ou adaptados, mantendo as mesmas verificações. Liste quais foram.

A sessão é longa. Mantenha um arquivo `tmp/refatoracao-impostos-progresso.md` atualizado com o que já foi feito, o próximo passo, as decisões tomadas e as divergências encontradas, para conseguir retomar o trabalho se perder contexto no meio.

Se não der para terminar tudo, prefira um estado parcial consistente a um estado completo quebrado: dois services migrados com a suíte verde valem mais do que quatro pela metade. Se terminar cedo, não amplie o escopo para outras refatorações em `billing/`. Use o tempo para reler o próprio diff com olhar crítico e reforçar os testes de borda, e anote no relatório as outras oportunidades que viu.

## Relatório

Escreva o relatório em `RELATORIO_REFATORACAO.md` na raiz do repositório e repita-o como mensagem final. Vou ler de manhã, sem lembrar dos detalhes e sem ter acompanhado nada, então ele precisa se sustentar sozinho. Abra com o estado: terminou ou não, suíte verde ou não. Depois:

- **O que mudou:** a nova estrutura em poucas linhas e como cada um dos quatro services passou a usá-la.
- **Divergências entre as quatro cópias:** para cada uma, onde está, o que difere, um exemplo de entrada em que os resultados diferem, e como você preservou a diferença.
- **Decisões que você tomou sozinho** e o motivo de cada uma.
- **Verificação:** quais comandos rodou, os resultados e a comparação com a linha de base. Separe o que você confirmou executando do que concluiu apenas lendo o código.
- **O que ficou pendente** ou precisa de decisão minha.
- **Como revisar:** nome da branch e a ordem sugerida de leitura dos commits.

Se algo não funcionou ou não foi verificado, diga isso diretamente. Prefiro um relatório que mostra os problemas a um que parece limpo.
