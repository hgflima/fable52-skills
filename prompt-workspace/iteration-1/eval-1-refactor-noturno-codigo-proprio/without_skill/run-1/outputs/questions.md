# Perguntas que eu faria antes de rodar

Não havia ninguém para responder, então segui com as suposições indicadas em cada item. O prompt funciona sem as respostas, mas fica melhor com elas.

1. **Quais são os quatro services?** Suposição: o agente descobre lendo `app/services/billing/`. Se você colar os nomes dos arquivos no prompt, ele economiza tempo e não corre o risco de escolher os quatro errados.
2. **Existem diferenças conhecidas e intencionais entre as quatro cópias** (arredondamento, alíquota, base de cálculo)? Suposição: não se sabe, então o prompt manda preservar tudo e listar as divergências para você decidir.
3. **O agente pode fazer push da branch e abrir PR?** Suposição: não. Fica tudo local até você revisar.
4. **Como se roda a suíte e o linter** (RSpec ou Minitest, `bin/rspec`, `bundle exec rubocop`, algum script de CI)? Suposição: o agente descobre pelo repositório.
5. **A suíte roda inteira localmente sem serviços externos?** Suposição: sim. Se ela depender de gateway de pagamento em sandbox ou de outro serviço, vale avisar no prompt o que pode e o que não pode ser chamado.
6. **Qual é a cobertura de testes atual dos quatro services?** Suposição: parcial, por isso o prompt pede testes de caracterização antes de refatorar.
7. **Há cálculo de imposto fora de `app/services/billing/` que também deveria entrar no escopo?** Suposição: não. O agente só registra o que encontrar.
8. **Migrations e gems novas estão liberadas?** Suposição: não, por ser uma execução sem supervisão. O agente propõe no relatório.
9. **Onde você quer o relatório?** Suposição: `RELATORIO_REFATORACAO.md` na raiz do repositório, mais a mensagem final da sessão.
