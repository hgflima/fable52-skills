# Perguntas que eu faria antes de escrever o prompt

Ninguém estava disponível para responder nesta execução. O prompt foi escrito com a suposição marcada em cada pergunta.

1. **Qual comando roda a suíte de testes do monolito?**
   - a) `bundle exec rspec`
   - b) `bin/rails test`
   - c) Outro (qual?)

   Suposição usada: nenhuma das três. O prompt manda o modelo usar o comando que a CI roda e descobrir isso pela configuração da CI e pela presença de `spec/` ou `test/`.

2. **Se as quatro cópias do cálculo de imposto derem resultados diferentes para a mesma entrada (arredondamento, alíquota, ordem das operações), o que o modelo deve fazer?**
   - a) Preservar o resultado atual de cada service, deixar a diferença explícita no código compartilhado e listar tudo no relatório
   - b) Unificar na versão de um service que você indicar como a correta (qual?)
   - c) Unificar na versão que parecer correta e relatar o que mudou

   Suposição usada: a). É código que define quanto o cliente paga, e decidir qual variante está certa é uma decisão fiscal, não de refactor.

3. **Como você quer encontrar o trabalho de manhã?**
   - a) Commits em um branch local novo, sem push
   - b) Branch enviado ao remoto com um pull request em rascunho
   - c) Alterações no working tree, sem commit

   Suposição usada: a), no branch `billing-tax-consolidation`, com um commit por service migrado.
