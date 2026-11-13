# Validação do Projeto #43

Verificação independente executada em ambiente Codex com Node.js 24.22.1. A API real foi importada e exercitada com as duas migrações reais em SQLite em memória por um adaptador da interface D1. Os testes não usam credenciais de produção.

## Testes automatizados

Dentro da subpasta do projeto, execute com Node.js 24 LTS.

```powershell
node --test tests/api.test.mjs
```

Resultado registrado — 8 testes aprovados, 0 falhas.

| Caso | Verificação |
| --- | --- |
| Fisher–Yates e equipes | Lista original preservada, nenhuma pessoa perdida ou repetida, limite de tamanho e equilíbrio entre equipes, ordem individual |
| Sessões | Cookie antigo falsificado rejeitado, token desconhecido e sessão expirada rejeitados, sessão válida recupera perfil |
| Autorização | Origem externa e ausente rejeitadas, criação exige login, payload inválido e JSON null rejeitados, ações exclusivas do organizador |
| Sorteio e ata | Entrada com Discord idempotente, dois pedidos simultâneos retornam uma única ata, hash conferido, consulta pública, repetição preserva resultado |
| Banco imutável | UPDATE e DELETE da ata bloqueados por triggers, alteração de participante após sorteio bloqueada |
| Regras de lista | Mínimo de dois nomes, tamanho máximo inteiro de 1 a 100, remoção limitada à sala informada |
| OAuth2 | Cookie de estado obrigatório, estado expirado ou reutilizado rejeitado, callback bem-sucedido com respostas Discord simuladas |
| Concorrência | Mudança da lista entre leitura e gravação rejeita sorteio com 409, sem publicar uma ata incompleta |

A sequência bem-sucedida do OAuth2 usa um mock somente nas duas respostas HTTP externas do Discord. O processamento do callback, SQL, criação da sessão e logout são executados pela implementação real.

## Verificador oficial

Na raiz do repositório, execute.

```powershell
node validar-local.mjs P26-2B-69DF-2DBB-E2A5-276B
```

Resultado registrado — 8 verificações estruturais aprovadas. O item de deploy online está pendente porque a URL real ainda não foi configurada no README. O verificador estrutural não substitui os testes funcionais ou o teste de login real.

As checagens `node --check public/app.js` e `node --check functions/api/[[path]].js` passaram. A revisão da interface confirmou uso de `textContent` para nomes, títulos e resultados recebidos da API.

## Pendências antes da entrega

- Configurar o UUID real do D1 e aplicar migrações na Cloudflare.
- Configurar as credenciais e o callback Discord.
- Publicar e testar OAuth2 com contas reais, inclusive a entrada de um aluno e a consulta da ata sem login.
- Atualizar a URL no README e repetir o verificador oficial.

A bateria automatizada não afirma disponibilidade na Cloudflare nem aprovação pela instituição. O teste de concorrência usa SQLite local e a lógica SQL real, não carga distribuída de produção. O hash comprova correspondência entre o conteúdo recebido e o resumo registrado, sem fornecer testemunho externo do sorteio.

## Integração adicional

As duas migrações foram aplicadas com sucesso no D1 local do Wrangler 4.161.0. A API compilou para Pages Functions. Requisições HTTP reais contra esse runtime criaram salas nos dois modos, publicaram atas, conferiram seus hashes, repetiram o pedido preservando a ata e consultaram o resultado sem sessão. Dez participantes formaram equipes de tamanhos 4, 3 e 3. A resposta estática também incluiu a política CSP configurada.

O JavaScript real da interface foi exercitado em um DOM de teste, conectado aos handlers reais e SQLite. O fluxo de criar sala, adicionar e remover participantes, sortear e consultar a ata publicamente passou sem erros de script. Este ensaio usa uma sessão de teste inserida apenas no banco local.

A tentativa de inspeção visual com Chromium foi bloqueada pela restrição de sockets do ambiente. Portanto, revisão visual em navegador real, tamanhos de tela e impressão precisam de conferência após abrir a aplicação. O teste do DOM não mede layout.
