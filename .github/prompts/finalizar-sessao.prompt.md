---
description: "Finaliza a sessão de adição de conhecimento: executa git commit com mensagem relevante ao conteúdo trabalhado e git push."
name: "Finalizar Sessão"
agent: "agent"
---

Finalize a sessão atual de adição de conhecimento neste repositório seguindo estes passos:

1. Verifique quais arquivos foram criados ou modificados nesta sessão com `git status`
2. Analise os arquivos alterados para entender os temas abordados
3. Monte uma mensagem de commit **curta e relevante** aos assuntos trabalhados:
   - Use o formato: `assunto: descrição curta` ou `assunto1, assunto2: descrição curta`
   - Exemplos: `context engineering: novo artigo`, `arquitetura: seção de tradeoffs`, `config: perfil do autor e regras de exemplos`
   - Em português, direto ao ponto — sem "adiciona", "cria", "atualiza" genéricos quando der para ser mais específico
4. Execute `git add .`
5. Execute `git commit -m "<mensagem>"`
6. Execute `git push`
7. Confirme que o push foi concluído com sucesso
