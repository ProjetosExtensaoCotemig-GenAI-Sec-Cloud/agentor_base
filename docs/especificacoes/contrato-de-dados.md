# Contrato de dados: documento e resultado de busca

**Versão:** 0.2 (rascunho com os ajustes da revisão do PR #2; aguardando a revisão das três frentes para virar 1.0)
**Task:** #24 · **História:** #22 B1 Preparar corpus e contrato de dados

Acordo sobre **campos e formatos** de duas coisas que todas as frentes usam: o **documento** de uma fonte de conhecimento e o **resultado de busca** que o assistente recebe. Com isso fixado, uma frente pode escrever fontes, outra a busca e outra os testes e avaliações, sem depender uma da outra.

O contrato descreve o que o código já faz (`packages/knowledge/knowledge/src/index.ts` e `packages/knowledge/tool-knowledge/src/index.ts`) e o que a [especificação da `read_document`](tool-read-document.md#32-extensão-do-contrato-knowledgesource) já previu. Tudo o que o contrato pede e o código ou a especificação ainda não têm está marcado como **proposta** ou **pendência de código** e reunido na [seção 5](#5-propostas-e-pendências).

## 1. Documento

Um documento é um item de uma fonte autorizada, como um arquivo Markdown na pasta `conhecimento/` de um cliente.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `sourceId` | texto | sim | **Origem:** id da fonte que contém o documento (ex.: `exemplo-manual`). É o mesmo id do [inventário de fontes](../fontes/inventario-de-fontes.md) e do `cordis.patch.yml` do cliente. **Proposta P2:** campo novo no objeto documento |
| `documentId` | texto | sim | **Identificador** estável do documento dentro da fonte. Na pasta local, é o caminho relativo à raiz da fonte, **sempre com `/`** (ex.: `manual-do-estagiario.md`, `sub/nested.md`). **Pendência de código C1:** no Windows, o provider ainda devolve `\` em subpastas |
| `title` | texto | sim | **Título** legível: o primeiro título Markdown (`# ...`) ou, sem ele, o `documentId` |
| `content` | texto | sim | **Texto** do documento, em UTF-8 |
| `version` | texto | não | **Versão**, quando a fonte tiver. Na pasta local, segue a regra 5 do [inventário](../fontes/inventario-de-fontes.md#3-regras): data da última alteração + os 12 primeiros caracteres do SHA-256 do arquivo. Em fontes remotas: ETag, revisão ou data de modificação |

A identidade de um documento é o **par `sourceId` + `documentId`**. O `documentId` sozinho pode se repetir em fontes diferentes.

**Relação com o código:** o documento parte do `KnowledgeDocument` da [especificação da `read_document`](tool-read-document.md#32-extensão-do-contrato-knowledgesource) (task #20, seção 3.2; implementação prevista na #41). Aquela especificação **não tem `sourceId`** no objeto: a origem fica na chamada `read(sourceId, documentId)`. Incluir `sourceId` no próprio documento é a **proposta P2**, a decidir pelas frentes antes da #41. Os campos técnicos `bytes` e `truncated` daquela especificação continuam valendo para a tool de leitura.

### Exemplo 1: documento do manual

```json
{
  "sourceId": "exemplo-manual",
  "documentId": "manual-do-estagiario.md",
  "title": "Manual do Estagiário — Cliente Exemplo",
  "content": "# Manual do Estagiário — Cliente Exemplo\n\nDocumento fictício usado para validar a base de conhecimento. Não descreve uma empresa real.\n\n## Jornada e ponto eletrônico\n\nA jornada do estagiário é de 6 horas diárias, 30 horas semanais. (...)",
  "version": "2026-11-13 sha256:71ff45624a04"
}
```

### Exemplo 2: documento da política

```json
{
  "sourceId": "exemplo-manual",
  "documentId": "politica-de-seguranca.md",
  "title": "Política de Segurança da Informação — resumo para estagiários",
  "content": "# Política de Segurança da Informação — resumo para estagiários\n\nDocumento fictício usado para validar a base de conhecimento.\n\n## Senhas\n\nSenhas são pessoais e intransferíveis, com no mínimo 12 caracteres. (...)",
  "version": "2026-11-13 sha256:63dddb885ff5"
}
```

No exemplo, `content` foi abreviado com `(...)`. Na prática, ele traz o texto inteiro ou o texto cortado no limite da tool.

## 2. Resultado de busca

### 2.1 Campos

| Campo | Tipo | Descrição |
|---|---|---|
| `query` | texto | **Pergunta** pesquisada, como foi enviada. **Proposta P1:** hoje o `KnowledgeSearchResult` do código não guarda a pergunta (ela fica só na chamada da tool). Incluí-la deixa o resultado autoexplicativo em logs e avaliações |
| `hits` | lista de trechos | **Trechos** encontrados, do mais para o menos relevante. Lista vazia = nada encontrado |
| `failures` | lista | Fontes que falharam, com `sourceId` e `message`. Uma fonte com erro não derruba a busca das outras |

Cada trecho (`KnowledgeHit`, já existente no código):

| Campo | Tipo | Descrição |
|---|---|---|
| `sourceId` | texto | **Fonte** do trecho |
| `documentId` | texto | Documento de origem do trecho dentro da fonte, com as mesmas regras do documento (seção 1) |
| `title` | texto | Título do documento de origem |
| `excerpt` | texto | Trecho relevante, cortado no limite da fonte (padrão de 800 caracteres na pasta local) |
| `score` | número | Relevância; maior é melhor. As fontes devem usar a escala de 0 a 1 |

### 2.2 Texto que o modelo recebe

A tool `knowledge_search` transforma o resultado em texto (função `renderResult`), um bloco por trecho, com a origem sempre explícita:

```
[<n>] <title> (fonte: <sourceId>, documento: <documentId>)
<excerpt>
```

Sem resultado, o texto é: `Nenhum trecho encontrado na base de conhecimento do cliente.`
Para cada fonte que falhou, entra uma linha: `Aviso: a fonte <sourceId> falhou: <message>` (veja o exemplo D).

### 2.3 Exemplos

Os quatro exemplos foram **gerados executando** o provider `@agentor/knowledge-fs` e a função `renderResult` sobre os documentos reais do cliente `exemplo`. Só três pontos foram ajustados à mão: o campo `query`, acrescentado conforme a proposta P1; o `score` do exemplo A, arredondado (0,9667 → 0,97); e a mensagem de falha do exemplo D, que mostra o texto esperado pelo contrato (pendência C2).

**Exemplo A — com resultado**

```json
{
  "query": "Como funciona o ponto eletrônico?",
  "hits": [
    {
      "sourceId": "exemplo-manual",
      "documentId": "manual-do-estagiario.md",
      "title": "Manual do Estagiário — Cliente Exemplo",
      "excerpt": "## Jornada e ponto eletrônico A jornada do estagiário é de 6 horas diárias, 30 horas semanais. O ponto eletrônico é registrado no aplicativo Ponto Exemplo na entrada, na saída para o intervalo, no retorno e na saída final. Esquecimentos de marcação devem ser justificados ao mentor no mesmo dia pelo formulário de ajuste de ponto.",
      "score": 0.97
    }
  ],
  "failures": []
}
```

Texto para o modelo:

```
[1] Manual do Estagiário — Cliente Exemplo (fonte: exemplo-manual, documento: manual-do-estagiario.md)
## Jornada e ponto eletrônico A jornada do estagiário é de 6 horas diárias, 30 horas semanais. (...)
```

**Exemplo B — com resultado em outro documento**

```json
{
  "query": "Qual a regra de senhas?",
  "hits": [
    {
      "sourceId": "exemplo-manual",
      "documentId": "politica-de-seguranca.md",
      "title": "Política de Segurança da Informação — resumo para estagiários",
      "excerpt": "## Senhas Senhas são pessoais e intransferíveis, com no mínimo 12 caracteres. Nunca compartilhe senhas por chat ou e-mail; use o cofre de senhas corporativo.",
      "score": 0.5
    }
  ],
  "failures": []
}
```

**Exemplo C — sem resultado**

```json
{
  "query": "Qual o valor do vale-refeição?",
  "hits": [],
  "failures": []
}
```

Texto para o modelo:

```
Nenhum trecho encontrado na base de conhecimento do cliente.
```

Nesse caso, o assistente deve dizer que não encontrou a informação e **não inventar** uma resposta. Essa regra já está na descrição da tool.

**Exemplo D — uma fonte falhou e a outra respondeu**

Cenário de teste: além de `exemplo-manual`, o cliente tem uma segunda fonte de pasta local, `exemplo-rh`, cuja pasta não existe. A busca continua na fonte que funciona, e a falha vira um item em `failures`.

```json
{
  "query": "Qual a regra de senhas?",
  "hits": [
    {
      "sourceId": "exemplo-manual",
      "documentId": "politica-de-seguranca.md",
      "title": "Política de Segurança da Informação — resumo para estagiários",
      "excerpt": "## Senhas Senhas são pessoais e intransferíveis, com no mínimo 12 caracteres. Nunca compartilhe senhas por chat ou e-mail; use o cofre de senhas corporativo.",
      "score": 0.5
    }
  ],
  "failures": [
    { "sourceId": "exemplo-rh", "message": "Fonte indisponível: a pasta configurada não foi encontrada." }
  ]
}
```

Texto para o modelo:

```
[1] Política de Segurança da Informação — resumo para estagiários (fonte: exemplo-manual, documento: politica-de-seguranca.md)
## Senhas Senhas são pessoais e intransferíveis, com no mínimo 12 caracteres. Nunca compartilhe senhas por chat ou e-mail; use o cofre de senhas corporativo.
Aviso: a fonte exemplo-rh falhou: Fonte indisponível: a pasta configurada não foi encontrada.
```

A `message` acima é a **esperada pelo contrato**. Hoje o código devolve a mensagem crua do sistema, com o caminho absoluto do host (`ENOENT: no such file or directory, scandir '/srv/agentor/clients/exemplo/rh'`), o que fere a regra 1 da seção 4. É a **pendência de código C2**.

## 3. Como relacionar uma fonte exibida ao documento de origem

Todo trecho exibido carrega `sourceId` e `documentId`. Para chegar ao documento:

1. **`sourceId` → fonte:** procure o id no [inventário de fontes](../fontes/inventario-de-fontes.md) (campo *Fonte no sistema*) ou no `cordis.patch.yml` do cliente (`config.id` da linha `knowledge-*`). Lá está a raiz da fonte.
2. **`documentId` → arquivo:** na pasta local, junte a raiz com o `documentId`.

Exemplo: o trecho do Exemplo A mostra `fonte: exemplo-manual, documento: manual-do-estagiario.md`.
- `exemplo-manual` → raiz `config/clients/exemplo/conhecimento/`
- arquivo: [`config/clients/exemplo/conhecimento/manual-do-estagiario.md`](../../config/clients/exemplo/conhecimento/manual-do-estagiario.md)

**Ressalva até a pendência C1:** documentos em subpastas, indexados no Windows, aparecem hoje com `\` (ex.: `sub\nested.md`). O caminho ainda leva ao arquivo, mas o mesmo documento tem ids diferentes em cada sistema.

O mesmo par (`sourceId`, `documentId`) é a entrada da futura tool `read_document`. Assim, o assistente também consegue abrir o documento inteiro a partir de um trecho.

## 4. Regras do contrato

1. Nenhum campo pode trazer caminho absoluto do host, URL de download, token ou credencial. Isso inclui as mensagens em `failures`. **Pendência de código C2:** o código atual ainda repassa a mensagem crua do erro (exemplo D).
2. `documentId` é estável: o mesmo documento mantém o mesmo id entre buscas **e entre sistemas operacionais**, sempre com `/`. **Pendência de código C1.**
3. Toda fonte nova usa os mesmos campos. Quem cria um provider (OneDrive, banco, vetorial) devolve `KnowledgeHit` com `sourceId` e `documentId` preenchidos.
4. Mudança de campo = nova versão deste contrato, revisada pelas três frentes.

## 5. Propostas e pendências

### 5.1 Propostas para as frentes decidirem

| Id | Proposta | Situação |
|---|---|---|
| P1 | Incluir `query` no resultado de busca (seção 2.1) | A decidir |
| P2 | Incluir `sourceId` no objeto documento, que não existe no `KnowledgeDocument` da #20 (seção 1). Decidir antes da #41 | A decidir |
| P3 | Confirmar a escala de `score` de 0 a 1 para os próximos providers. A pasta local já segue essa escala | A decidir |
| — | Versão na pasta local: data + hash do conteúdo | A decidir (a regra 5 do inventário de fontes já usa esse formato) |

### 5.2 Pendências de código (fora do escopo desta task)

| Id | Pendência | Onde | O que o contrato espera |
|---|---|---|---|
| C1 | `documentId` usa o separador do sistema: `relative(root, file)` devolve `\` no Windows (testado com `sub\nested.md`) | `packages/knowledge/knowledge-fs/src/index.ts` | Normalizar para `/` (ex.: `relative(root, file).split(sep).join('/')`) e cobrir com teste |
| C2 | A mensagem de falha de uma fonte vai crua para o modelo, com o caminho absoluto do host (ex.: `ENOENT ... scandir '/srv/...'` quando a pasta não existe). O `KnowledgeRegistry` repassa `error.message` em `failures` e o `renderResult` exibe | `packages/knowledge/knowledge/src/index.ts` e providers | Mensagem genérica sem caminho (ex.: `Fonte indisponível: a pasta configurada não foi encontrada.`), com o detalhe só no log |

## 6. Revisões

| Frente | Revisor | Data | Resultado |
|---|---|---|---|
| A | | | |
| B | | | |
| C | | | |

Revisão técnica do PR #2 (não substitui a revisão por frente): Felipe Vandevelde, 2026-11-25, ajustes solicitados e aplicados na versão 0.2.
