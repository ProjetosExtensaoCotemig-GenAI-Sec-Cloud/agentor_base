# Entrega — task #33: Mapear três riscos e seus testes

| Campo | Valor |
|---|---|
| Frente | C |
| História | #29 C1 Definir necessidade e primeiro cenário |
| Sprint | 1: Escolher a base e definir o cenário |
| Objetivo | Antecipar as falhas de maior risco e definir como testá-las com segurança |
| Artefato | [Plano de testes de risco](../especificacoes/riscos-e-testes.md) |

## Critérios de aceite e evidências

| # | Critério | Evidência |
|---|---|---|
| 1 | O plano cobre acesso a arquivo fora da pasta, instrução maliciosa em documento e credencial exposta nos registros | [R1](../especificacoes/riscos-e-testes.md#r1--acesso-a-arquivo-fora-da-pasta-autorizada), [R2](../especificacoes/riscos-e-testes.md#r2--instrução-maliciosa-em-documento-injeção-por-documento) e [R3](../especificacoes/riscos-e-testes.md#r3--credencial-exposta-nos-registros), com a superfície de cada um no projeto hoje |
| 2 | Cada risco tem uma tentativa fictícia, o comportamento esperado e a evidência a coletar | Tabela por risco com as colunas *Tentativa fictícia*, *Comportamento esperado* e *Evidência a coletar*, e o [resumo](../especificacoes/riscos-e-testes.md#3-resumo) dos três |
| 3 | O ambiente e os limites do teste estão confirmados, e um marcador inventado substitui qualquer segredo real | [1.1 Ambiente](../especificacoes/riscos-e-testes.md#11-ambiente-confirmado-em-2026-11-25-no-repositório), com onde cada item foi confirmado; [1.3 Limites](../especificacoes/riscos-e-testes.md#13-limites) (seis regras); [1.2 Marcador](../especificacoes/riscos-e-testes.md#12-marcador-no-lugar-de-segredo) `AGENTOR-MARCADOR-FICTICIO-33`, usado nos três riscos |

## Como foi verificado

- As ferramentas disponíveis ao modelo foram conferidas nos patches dos bundles `kernel` e `training`: só a `knowledge_search`.
- O `.gitignore` foi conferido para `.env`, `.env.*` e `.dsh-home/`.
- Para R1a, o comportamento do `readdir` recursivo com links simbólicos foi testado à parte (Node 22.23, Linux): o link aparece como link, e a pasta apontada não é percorrida. A confirmação no Node 24 e com *junction* no Windows faz parte da execução.
- Nenhum teste de risco foi executado nesta task: o entregável é o plano.

## A preencher

| Campo | Valor |
|---|---|
| Responsável | Gabriella Isa Rodrigues Crisostomo |
| Revisor | |
| Dependências e alternativa com dados simulados | R1b depende da tool `read_document` (#41). R2b é opcional e exige LLM real com a chave do testador; sem ela, R2a roda offline. A referência à pendência C2 aponta para o contrato de dados (#24, PR #2) |
| Dados utilizados e risco | Só dados fictícios e o marcador inventado. Risco baixo: é um documento, e o plano proíbe segredos reais, links para pastas do sistema e commit da pasta de teste |
| Links do artefato, da revisão e da evidência | [Plano](../especificacoes/riscos-e-testes.md); link do PR. Revisão: |
| Horas realizadas | 2,5 h |

## Fora do escopo desta task

- Executar os testes e registrar os resultados (task de teste da Frente C).
- Corrigir falhas que a execução encontrar.
