# Inventário de fontes permitidas

Registro de **onde vêm os documentos** que o assistente pode consultar e **em que condições** podem ser usados. Só entra na base de conhecimento de um cliente a fonte que estiver neste inventário com situação **Autorizada**.

Atualizado em: 2026-11-25 (task #23).

## 1. Fontes autorizadas

| Campo | Fonte 1 | Fonte 2 |
|---|---|---|
| **Nome** | Manual do Estagiário — Cliente Exemplo | Política de Segurança da Informação — resumo para estagiários |
| **Arquivo** | `manual-do-estagiario.md` | `politica-de-seguranca.md` |
| **Autoria** | Felipe Vandevelde (equipe do projeto, Frente A) | Felipe Vandevelde (equipe do projeto, Frente A) |
| **Origem** | Criado pela equipe para o projeto. **Fictício**: não descreve empresa real | Criado pela equipe para o projeto. **Fictício**: não descreve empresa real |
| **Finalidade** | Validar a busca na base de conhecimento (jornada, ponto eletrônico, rotinas do estagiário) | Validar a busca na base de conhecimento (regras de senha e segurança) |
| **Versão** | 2026-11-13 · SHA-256 `71ff45624a04` · 933 bytes | 2026-11-13 · SHA-256 `63dddb885ff5` · 488 bytes |
| **Licença** | MIT, a mesma do repositório ([LICENSE](../../LICENSE)) | MIT, a mesma do repositório ([LICENSE](../../LICENSE)) |
| **Local de armazenamento** | [`config/clients/exemplo/conhecimento/`](../../config/clients/exemplo/conhecimento/manual-do-estagiario.md) | [`config/clients/exemplo/conhecimento/`](../../config/clients/exemplo/conhecimento/politica-de-seguranca.md) |
| **Fonte no sistema** | `exemplo-manual` (provider `@agentor/knowledge-fs`, em [`cordis.patch.yml`](../../config/clients/exemplo/cordis.patch.yml)) | `exemplo-manual` (mesma fonte) |
| **Dados pessoais** | Nenhum | Nenhum |
| **Material fictício** | **Sim**, declarado logo abaixo do título do arquivo | **Sim**, declarado logo abaixo do título do arquivo |
| **Situação** | Autorizada | Autorizada |

## 2. Fontes externas propostas (pendentes)

Qualquer dado que **não** tenha sido criado pela equipe (documento de uma empresa parceira, material da faculdade, página da web, pasta do OneDrive ou do Google Drive etc.) entra aqui primeiro, com situação **Pendente**. Ele só passa para a seção 1 quando houver **autorização registrada**: quem autorizou, quando e onde está o registro (e-mail, comentário no Taiga ou termo).

| Nome | Origem | Proposto por | Finalidade | Autorização (quem, quando, onde) | Situação |
|---|---|---|---|---|---|
| — | — | — | — | — | Nenhuma fonte externa proposta até 2026-11-18 |

## 3. Regras

1. **Fonte nova = linha nova.** Quem adicionar um documento a `config/clients/<cliente>/conhecimento/` registra a fonte neste inventário no mesmo commit.
2. **Fictício marcado duas vezes:** logo abaixo do título do próprio documento ("Documento fictício…") e no campo *Material fictício* acima.
3. **Externo começa pendente.** Sem autorização registrada, o documento não entra no repositório nem na base de conhecimento. Enquanto isso, use um documento fictício no lugar.
4. **Sem dados pessoais nem segredos** em nenhuma fonte, fictícia ou autorizada.
5. **Versão = data + hash do conteúdo.** A versão de uma fonte é a data da última alteração do documento mais os 12 primeiros caracteres do SHA-256 do arquivo **como fica no Git, com finais de linha LF**, em minúsculas. Calcule depois do `git add` e antes do commit, no Git Bash ou Linux: `git show :<arquivo> | sha256sum`. Não use o arquivo da pasta de trabalho no Windows: com finais de linha CRLF, o hash dá outro valor. O hash existe antes do commit, então o registro cabe no mesmo commit do documento (regra 1) e não muda com squash ou rebase. Ao alterar um documento, atualize a data e o hash da linha correspondente.
