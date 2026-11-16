# Perguntas para o usuário

1. Qual é o motivo da regra "nunca apagar arquivos"? Aconteceu algum incidente específico, ou é uma precaução geral? A regra vale também para arquivos temporários que o próprio agente cria?
2. Quais testes do checkout estão flaky, com que frequência falham, e isso acontece só no CI ou também localmente?
3. Qual comando roda a suíte de testes do checkout?
4. O agente pode alterar código de produção do checkout quando a causa do flaky for um bug real, ou só os testes?
5. O limite de 200.000 tokens é uma restrição real de custo, ou só estava ali para o modelo "se organizar"?
6. O ambiente onde o prompt roda tem subagentes disponíveis? Em caso afirmativo, por que o prompt original pedia para usá-los em tudo?
7. A regra de frases com menos de 15 palavras era para o relatório final ou para tudo, incluindo comentários de código? Qual era o objetivo: respostas curtas?
8. Esse texto vai como system prompt ou como mensagem do usuário? Você precisa ver o raciocínio do modelo durante a execução?

# Suposições adotadas na ausência de resposta

- A regra de não apagar arquivos é uma precaução geral e vale para arquivos existentes do repositório, não para temporários criados pelo agente.
- O agente pode corrigir código de produção quando a mudança for pequena e contida.
- O limite de tokens não é uma restrição real e foi removido.
- Subagentes estão disponíveis, mas devem ser usados só para trabalho paralelo e independente.
- A regra das 15 palavras servia para obter respostas curtas e foi trocada por um formato de relatório final.
- O prompt foi mantido em inglês, como o original.
