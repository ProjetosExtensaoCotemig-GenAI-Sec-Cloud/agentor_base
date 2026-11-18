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
| 2 | Há exemplos de resultado de busca com pergunta, trechos e fontes, incluindo o caso sem resultado | [Seção 2.3](../especificacoes/contrato-de-dados.md#23-exemplos): exemplos A e B com resultado e exemplo C sem resultado, cada um com `query`, `hits` e `sourceId`, mais o texto que o modelo recebe |
| 3 | As três frentes revisaram o rascunho antes de fechar a versão inicial | **Pendente.** O rascunho está na versão 0.1, com os pontos para decisão na [seção 5](../especificacoes/contrato-de-dados.md#5-pontos-para-a-revisão-das-frentes) e a tabela de revisões na [seção 6](../especificacoes/contrato-de-dados.md#6-revisões). Ao fim da revisão, a versão passa para 1.0 |
| 4 | É possível relacionar cada fonte exibida ao documento de origem | [Seção 3](../especificacoes/contrato-de-dados.md#3-como-relacionar-uma-fonte-exibida-ao-documento-de-origem): o par `sourceId` + `documentId` leva ao inventário, à raiz da fonte e ao arquivo, com um exemplo resolvido até o arquivo real |

## Como foi verificado

- Os exemplos de resultado de busca foram **gerados executando** o provider `@agentor/knowledge-fs` e a função `renderResult` da `knowledge_search` sobre `config/clients/exemplo/conhecimento/`, com as perguntas "Como funciona o ponto eletrônico?", "Qual a regra de senhas?" e "Qual o valor do vale-refeição?". O único campo acrescentado à mão foi `query`, marcado como proposta.
- Os campos do documento foram conferidos com o `KnowledgeHit` do código e com o `KnowledgeDocument` da especificação da `read_document`.
- A versão dos documentos (`a878239`) foi tirada do `git log` de cada arquivo.

## A preencher

| Campo | Valor |
|---|---|
| Responsável | Gabriella Isa Rodrigues Crisostomo|
| Revisor | Um de cada frente (A, B e C); veja a seção 6 do contrato |
| Dependências e alternativa com dados simulados | Depende do inventário da #23 (ids das fontes). Usa só os documentos fictícios do cliente `exemplo` |
| Dados utilizados e risco | Somente documentos fictícios. Risco baixo: nenhum dado pessoal, nenhuma credencial, nenhum código alterado |
| Links do artefato, da revisão e da evidência | [Contrato](../especificacoes/contrato-de-dados.md); link do PR. Revisão: |
| Horas realizadas | 1 h |

## Fora do escopo desta task

- Implementar o campo `query` no código: só depois que as frentes aprovarem a proposta.
- Tool `read_document`: task #41.
