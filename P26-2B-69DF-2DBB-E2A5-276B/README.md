# Projeto #43 — Sorteador Auditável

Ferramenta para criar salas, formar equipes e definir a ordem de apresentação com ata pública.

- Estudante — Beatrys Belo de Lira Esteves
- RA — 2026108406
- Turma — 2B
- Chave Única — `P26-2B-69DF-2DBB-E2A5-276B`
- URL_CLOUDFLARE — pendente de publicação. Preencha com a URL real antes da entrega.

## Funcionamento

O organizador entra pelo Discord e cria uma sala com título, modalidade e limite de pessoas por equipe. Pode cadastrar nomes manualmente e compartilhar o link para que alunos entrem com Discord. Cada conta entra uma vez na sala. O organizador pode remover participantes antes do sorteio.

O servidor executa Fisher–Yates com Web Crypto e rejeição de valores para evitar viés na escolha dos índices. Ao sortear, a sala é encerrada. Uma única ata é gravada com a lista usada, resultado, organizador, horário UTC, código público e SHA-256 do JSON exato do registro. Repetir a requisição devolve a ata já salva. As equipes são distribuídas de forma equilibrada respeitando o limite escolhido.

Qualquer pessoa com o link pode consultar a ata e conferir seu hash. Nomes e IDs de Discord incluídos na ata ficam públicos para quem tiver esse link. O hash permite detectar alteração do conteúdo em relação ao hash registrado. A proteção de imutabilidade é aplicada por triggers SQLite e pelas rotas da aplicação. Um administrador que controle o banco e o código ainda pode alterar a infraestrutura. Este projeto não fornece prova externa da aleatoriedade ou do momento da publicação.

## Arquitetura

- `public/` — HTML5, CSS3 e JavaScript puro, com impressão da ata.
- `functions/` — API Cloudflare Pages Functions e login Discord OAuth2.
- `lib/` — utilitários do servidor e algoritmo de sorteio.
- `migrations/0001_init.sql` — estrutura original preservada para instalações existentes.
- `migrations/0002_sorteador.sql` — salas, participantes, sessões e atas.
- `tests/` — testes independentes de regras, autenticação e integridade.
- `AGENTS.md` e `skills/` — registro do trabalho dos agentes.

## Preparar a Cloudflare

Dentro desta pasta, abra um terminal. No PowerShell do laboratório use `npx.cmd` para evitar bloqueios de scripts `.ps1`.

```powershell
npx.cmd wrangler login
npx.cmd wrangler d1 create db-p26-2b-69df-2dbb-e2a5-276b
```

Copie o `database_id` retornado para `wrangler.jsonc`, substituindo o UUID composto de zeros. Para um banco já existente, use o ID correto dele. Aplique todas as migrações sem apagar o banco.

```powershell
npx.cmd wrangler d1 migrations apply DB --remote
```

No Cloudflare Pages, conecte o fork `Beatrys-lira/Avalia2026-2` com estas opções.

| Campo | Valor |
| --- | --- |
| Diretório raiz | `P26-2B-69DF-2DBB-E2A5-276B` |
| Comando de build | vazio |
| Diretório de saída | `public` |
| Branch de produção | `main` |
| Binding D1 | `DB`, apontando para o banco criado |

Cadastre os secrets `DISCORD_CLIENT_ID` e `DISCORD_CLIENT_SECRET` nas configurações do projeto Pages. O Client Secret deve ser inserido somente no painel ou no arquivo local `.dev.vars`, que é ignorado pelo Git.

No [Discord Developer Portal](https://discord.com/developers/applications), abra a aplicação e registre no OAuth2 o callback com o domínio real do site, seguido de `/api/auth/discord/callback`. O aplicativo solicita apenas o escopo `identify`.

Depois de configurar, faça novo deploy. Teste login, criação de sala, entrada de outra conta, sorteio e consulta da ata sem login. Preencha `URL_CLOUDFLARE` acima com o domínio real. A URL ainda não foi publicada nem validada nesta versão.

## Executar localmente

```powershell
npx.cmd wrangler d1 migrations apply DB --local
npx.cmd wrangler pages dev public
```

Abra o endereço HTTP mostrado pelo Wrangler, normalmente `http://localhost:8788`. O login local requer também esse callback cadastrado no Discord e as duas credenciais em `.dev.vars`.

```text
DISCORD_CLIENT_ID=ID_DA_APLICACAO
DISCORD_CLIENT_SECRET=SEGREDO_DA_APLICACAO
```

Use o mesmo hostname no navegador e no callback. As funções precisam do servidor Wrangler para funcionar.

## Validar e entregar

Consulte `VALIDACAO.md` para os resultados e limitações dos testes. Dentro desta pasta, com Node.js 24 LTS ou superior, execute a bateria funcional.

```powershell
node --test tests/api.test.mjs
```

Na raiz do repositório execute o validador oficial.

```powershell
node validar-local.mjs P26-2B-69DF-2DBB-E2A5-276B
```

O teste de disponibilidade online só pode passar após publicar e preencher a URL real. Envie apenas alterações desta subpasta no Pull Request para `frankalcantara/Avalia2026-2`, após a validação final. No AVA, informe a URL pública e o endereço do fork.

## Referências técnicas

- [Cloudflare Pages Functions](https://developers.cloudflare.com/pages/functions/)
- [Banco D1 e operações em lote](https://developers.cloudflare.com/d1/worker-api/d1-database/)
- [OAuth2 do Discord](https://docs.discord.com/developers/topics/oauth2)

Licença MIT.
