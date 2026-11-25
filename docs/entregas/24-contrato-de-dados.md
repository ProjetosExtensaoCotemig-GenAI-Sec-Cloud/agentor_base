# Entrega — task #24: Definir contrato de documento e resultado de busca

| Campo | Valor |
|---|---|
| Frente | B |
| História | #22 B1 Preparar corpus e contrato de dados |
| Sprint | 1: Escolher a base e definir o cenário |
| Objetivo | Permitir que as frentes trabalhem em paralelo com um acordo claro sobre campos e formatos |
| Artefato | [Contrato de dados](../especificacoes/contrato-de-dados.md) |

## Critérios de aceite e evidências

| # | Critério | Evidência |
|---|---|---|
| 1 | Há exemplos de documento com identificador, título, texto, origem e versão | [Seção 1](../especificacoes/contrato-de-dados.md#1-documento): tabela de campos (`documentId`, `title`, `content`, `sourceId`, `version`) e dois exemplos com os documentos reais do cliente `exemplo` |
| 2 | Há exemplos de resultado de busca com pergunta, trechos e fontes, incluindo o caso sem resultado | [Seção 2.3](../especificacoes/contrato-de-dados.md#23-exemplos): exemplos A e B com resultado, C sem resultado e D com uma fonte em falha, cada um com `query`, `hits` e `sourceId`, mais o texto que o modelo recebe |
| 3 | As três frentes revisaram o rascunho antes de fechar a versão inicial | **Pendente.** O rascunho está na versão 0.2, com os pontos para decisão na [seção 5](../especificacoes/contrato-de-dados.md#5-propostas-e-pendências) e a tabela de revisões na [seção 6](../especificacoes/contrato-de-dados.md#6-revisões). Ao fim da revisão, a versão passa para 1.0. A revisão técnica do PR #2 (Felipe Vandevelde) não conta como revisão de frente |
| 4 | É possível relacionar cada fonte exibida ao documento de origem | [Seção 3](../especificacoes/contrato-de-dados.md#3-como-relacionar-uma-fonte-exibida-ao-documento-de-origem): o par `sourceId` + `documentId` leva ao inventário, à raiz da fonte e ao arquivo, com um exemplo resolvido até o arquivo real. **Ressalva:** no Windows, documentos em subpastas ainda saem com `\` (pendência de código C1, [seção 5.2](../especificacoes/contrato-de-dados.md#52-pendências-de-código-fora-do-escopo-desta-task)) |

## Como foi verificado

- Os exemplos de resultado de busca foram **gerados executando** o provider `@agentor/knowledge-fs` e a função `renderResult` da `knowledge_search` sobre `config/clients/exemplo/conhecimento/`, com as perguntas "Como funciona o ponto eletrônico?", "Qual a regra de senhas?" e "Qual o valor do vale-refeição?". Foram escritos à mão só o campo `query`, marcado como proposta, e a mensagem de falha do exemplo D, que mostra o texto esperado pelo contrato (pendência C2).
- Os campos do documento foram conferidos com o `KnowledgeHit` do código e com o `KnowledgeDocument` da especificação da `read_document`.
- A versão dos documentos segue a regra 5 do inventário: data do `git log` + os 12 primeiros caracteres do `sha256sum` de cada arquivo.
- O exemplo D foi gerado com uma segunda fonte apontando para uma pasta inexistente. A mensagem crua do código (com caminho absoluto) foi registrada como pendência C2, e o exemplo mostra a mensagem esperada pelo contrato.

## A preencher

| Campo | Valor |
|---|---|
| Responsável | Gabriella Isa Rodrigues Crisostomo |
| Revisor | Um de cada frente (A, B e C); veja a seção 6 do contrato |
| Dependências e alternativa com dados simulados | Depende do inventário da #23 (ids das fontes e regra de versão); entra depois do PR #1. Usa só os documentos fictícios do cliente `exemplo` |
| Dados utilizados e risco | Somente documentos fictícios. Risco baixo: nenhum dado pessoal, nenhuma credencial, nenhum código alterado |
| Links do artefato, da revisão e da evidência | [Contrato](../especificacoes/contrato-de-dados.md); link do PR. Revisão: |
| Horas realizadas | 1 h |

## Fora do escopo desta task

- Implementar as propostas P1 a P3 no código: só depois que as frentes decidirem.
- Corrigir as pendências de código C1 (`documentId` com `/` no Windows) e C2 (mensagem de falha sem caminho absoluto).
- Tool `read_document`: task #41.
