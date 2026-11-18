# Contrato de dados: documento e resultado de busca

**Versão:** 0.1 (rascunho, aguardando revisão das três frentes)
**Task:** #24 · **História:** #22 B1 Preparar corpus e contrato de dados

Acordo sobre **campos e formatos** de duas coisas que todas as frentes usam: o **documento** de uma fonte de conhecimento e o **resultado de busca** que o assistente recebe. Com isso fixado, uma frente pode escrever fontes, outra a busca e outra os testes e avaliações, sem depender uma da outra.

O contrato descreve o que o código já faz (`packages/knowledge/knowledge/src/index.ts` e `packages/knowledge/tool-knowledge/src/index.ts`) e o que a [especificação da `read_document`](tool-read-document.md#32-extensão-do-contrato-knowledgesource) já previu. Onde o contrato pede algo que o código ainda não tem, isso está marcado como **proposta**.

## 1. Documento

Um documento é um item de uma fonte autorizada, como um arquivo Markdown na pasta `conhecimento/` de um cliente.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `sourceId` | texto | sim | **Origem:** id da fonte que contém o documento (ex.: `exemplo-manual`). É o mesmo id do [inventário de fontes](../fontes/inventario-de-fontes.md) e do `cordis.patch.yml` do cliente |
| `documentId` | texto | sim | **Identificador** estável do documento dentro da fonte. Na pasta local, é o caminho relativo à raiz da fonte, com `/` (ex.: `manual-do-estagiario.md`) |
| `title` | texto | sim | **Título** legível: o primeiro título Markdown (`# ...`) ou, sem ele, o `documentId` |
| `content` | texto | sim | **Texto** do documento, em UTF-8 |
| `version` | texto | não | **Versão**, quando a fonte tiver: commit, ETag, revisão ou data de modificação. Na pasta local do repositório, o commit que alterou o arquivo por último |

A identidade de um documento é o **par `sourceId` + `documentId`**. O `documentId` sozinho pode se repetir em fontes diferentes.

**Relação com o código:** é o `KnowledgeDocument` da [especificação da `read_document`](tool-read-document.md#32-extensão-do-contrato-knowledgesource) (implementação prevista na #41), com `sourceId` incluído no próprio objeto. Os campos técnicos `bytes` e `truncated` daquela especificação continuam valendo para a tool de leitura.

### Exemplo 1: documento do manual

```json
{
  "sourceId": "exemplo-manual",
  "documentId": "manual-do-estagiario.md",
  "title": "Manual do Estagiário — Cliente Exemplo",
  "content": "# Manual do Estagiário — Cliente Exemplo\n\nDocumento fictício usado para validar a base de conhecimento. Não descreve uma empresa real.\n\n## Jornada e ponto eletrônico\n\nA jornada do estagiário é de 6 horas diárias, 30 horas semanais. (...)",
  "version": "a878239"
}
```

### Exemplo 2: documento da política

```json
{
  "sourceId": "exemplo-manual",
  "documentId": "politica-de-seguranca.md",
  "title": "Política de Segurança da Informação — resumo para estagiários",
  "content": "# Política de Segurança da Informação — resumo para estagiários\n\nDocumento fictício usado para validar a base de conhecimento.\n\n## Senhas\n\nSenhas são pessoais e intransferíveis, com no mínimo 12 caracteres. (...)",
  "version": "a878239"
}
```

No exemplo, `content` foi abreviado com `(...)`. Na prática, ele traz o texto inteiro ou o texto cortado no limite da tool.

## 2. Resultado de busca

### 2.1 Campos

| Campo | Tipo | Descrição |
|---|---|---|
| `query` | texto | **Pergunta** pesquisada, como foi enviada. **Proposta:** hoje o `KnowledgeSearchResult` do código não guarda a pergunta (ela fica só na chamada da tool). Incluí-la deixa o resultado autoexplicativo em logs e avaliações |
| `hits` | lista de trechos | **Trechos** encontrados, do mais para o menos relevante. Lista vazia = nada encontrado |
| `failures` | lista | Fontes que falharam, com `sourceId` e `message`. Uma fonte com erro não derruba a busca das outras |

Cada trecho (`KnowledgeHit`, já existente no código):

| Campo | Tipo | Descrição |
|---|---|---|
| `sourceId` | texto | **Fonte** do trecho |
| `documentId` | texto | Documento de origem do trecho dentro da fonte |
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
Para cada fonte que falhou, entra uma linha: `Aviso: a fonte <sourceId> falhou: <message>`.

### 2.3 Exemplos

Os três exemplos foram **gerados executando** o provider `@agentor/knowledge-fs` e a função `renderResult` sobre os documentos reais do cliente `exemplo`. Só o campo `query` foi acrescentado, conforme a proposta, e o `score` do exemplo A foi arredondado (0,9667 → 0,97).

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

## 3. Como relacionar uma fonte exibida ao documento de origem

Todo trecho exibido carrega `sourceId` e `documentId`. Para chegar ao documento:

1. **`sourceId` → fonte:** procure o id no [inventário de fontes](../fontes/inventario-de-fontes.md) (campo *Fonte no sistema*) ou no `cordis.patch.yml` do cliente (`config.id` da linha `knowledge-*`). Lá está a raiz da fonte.
2. **`documentId` → arquivo:** na pasta local, junte a raiz com o `documentId`.

Exemplo: o trecho do Exemplo A mostra `fonte: exemplo-manual, documento: manual-do-estagiario.md`.
- `exemplo-manual` → raiz `config/clients/exemplo/conhecimento/`
- arquivo: [`config/clients/exemplo/conhecimento/manual-do-estagiario.md`](../../config/clients/exemplo/conhecimento/manual-do-estagiario.md)

O mesmo par (`sourceId`, `documentId`) é a entrada da futura tool `read_document`. Assim, o assistente também consegue abrir o documento inteiro a partir de um trecho.

## 4. Regras do contrato

1. Nenhum campo pode trazer caminho absoluto do host, URL de download, token ou credencial.
2. `documentId` é estável: o mesmo documento mantém o mesmo id entre buscas.
3. Toda fonte nova usa os mesmos campos. Quem cria um provider (OneDrive, banco, vetorial) devolve `KnowledgeHit` com `sourceId` e `documentId` preenchidos.
4. Mudança de campo = nova versão deste contrato, revisada pelas três frentes.

## 5. Pontos para a revisão das frentes

- **Campo `query` no resultado** (seção 2.1): aceitar a proposta e incluir no código, ou manter a pergunta só fora do resultado?
- **`version` na pasta local:** usar o commit do Git (como nos exemplos) ou a data de modificação do arquivo?
- **Escala de `score`:** o provider de pasta local já devolve valores de 0 a 1. Confirmar que os próximos providers seguem a mesma escala.

## 6. Revisões

| Frente | Revisor | Data | Resultado |
|---|---|---|---|
| A | | | |
| B | | | |
| C | | | |
