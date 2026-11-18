# Entrega — task #23: Registrar inventário de fontes permitidas

| Campo | Valor |
|---|---|
| Frente | B |
| História | #22 B1 Preparar corpus e contrato de dados |
| Sprint | 1: Escolher a base e definir o cenário |
| Objetivo | Registrar de onde vêm os documentos usados no projeto e em que condições podem ser usados |
| Artefato | [Inventário de fontes permitidas](../fontes/inventario-de-fontes.md) |

## Critérios de aceite e evidências

| # | Critério | Evidência |
|---|---|---|
| 1 | Cada fonte tem nome, autoria, origem, finalidade, versão, licença e local de armazenamento | [Seção 1](../fontes/inventario-de-fontes.md#1-fontes-autorizadas): uma coluna por fonte, com os sete campos preenchidos para os dois documentos do cliente `exemplo` |
| 2 | Os materiais fictícios estão marcados como tal | Campo *Material fictício* = **Sim** nas duas fontes, e a regra 2 ([seção 3](../fontes/inventario-de-fontes.md#3-regras)) exige a marcação também na primeira linha do documento, o que os dois arquivos já fazem |
| 3 | Qualquer dado externo proposto fica pendente até haver autorização registrada | [Seção 2](../fontes/inventario-de-fontes.md#2-fontes-externas-propostas-pendentes): tabela de fontes pendentes, com a coluna de autorização (quem, quando, onde). Nenhuma fonte externa foi proposta até agora. A regra 3 impede o uso antes da autorização |

## Como foi verificado

- As fontes listadas são todas as que existem em `config/clients/*/conhecimento/` no repositório (só o cliente `exemplo`, com dois arquivos).
- Autoria, commit e data foram tirados do `git log` de cada arquivo. O tamanho foi conferido com `wc -c`.
- A marcação "Documento fictício" foi conferida na primeira linha de cada arquivo.

## A preencher

| Campo | Valor |
|---|---|
| Responsável | Gabriella Isa Rodrigues Crisostomo|
| Revisor | |
| Dependências e alternativa com dados simulados | Nenhuma dependência: é um documento. Usa só os documentos fictícios já existentes |
| Dados utilizados e risco | Somente documentos fictícios. Risco baixo: nenhum dado pessoal, nenhuma credencial, nenhum código alterado |
| Links do artefato, da revisão e da evidência | [Inventário](../fontes/inventario-de-fontes.md); commit deste documento. Revisão: |
| Horas realizadas | 1 h |

## Fora do escopo desta task

- **Task #25:** as duas fontes sintéticas novas. Quando forem criadas, entram como linhas novas na seção 1 do inventário.
- **Task #24:** contrato de documento e resultado de busca.
