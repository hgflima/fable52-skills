# Perguntas que eu faria antes de escrever o prompt

Ninguém estava disponível para responder nesta execução. Escrevi o prompt com as suposições marcadas abaixo em cada pergunta.

1. **Que decisão essa análise precisa apoiar?**
   - a) Entender onde e quando perdemos clientes, para priorizar ações de retenção *(suposição usada)*
   - b) Identificar quais clientes atuais estão em risco de cancelar (modelo preditivo)
   - c) Medir o churn e acompanhar a evolução (taxa por mês, por coorte, por segmento)
   - d) Avaliar se uma ação já feita (mudança de preço, de onboarding, campanha) reduziu o churn

2. **Onde estão os dados e como o modelo vai acessá-los?**
   - a) Arquivos (CSV, planilha, Parquet) em uma pasta, com o Claude Code rodando nela *(suposição usada, com o caminho a preencher)*
   - b) Banco de dados ou data warehouse, consultado por SQL
   - c) Vou colar ou anexar os dados na conversa

3. **Quem vai ler o resultado, e em que formato?**
   - a) Liderança de produto e de customer success: relatório curto em português, com achados e recomendações *(suposição usada)*
   - b) Eu mesmo ou um time de dados: notebook com números, tabelas e código
   - c) Diretoria: resumo de uma página, ou slides
