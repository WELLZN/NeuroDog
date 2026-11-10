
# NeuroDog Internato

SaaS para gestão do centro de treinamento (CT) de cães do NeuroDog Internato: cães internados,
responsáveis, rotina diária por atividade e ocorrências. Mobile first, instalável como PWA.

## Telas

| Rota                                         | Tela                                                                                            |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `/login`                                     | Login individual                                                                                |
| `/tarefas`                                   | Tarefas do dia: meu painel, progresso das demandas, minhas responsabilidades e cães do CT       |
| `/caes`                                      | Cães do CT: busca, filtros (Atenção, Medicação, Saindo) e progresso da internação               |
| `/caes/:id`                                  | Perfil do cão: atenção ao manejar, progresso, abas Rotina/Dados/Histórico, registrar ocorrência |
| `/caes/novo`                                 | Entrada de cão novo (com fotos)                                                                 |
| `/tarefas/central`                           | Central de tarefas: adicionar tarefa (editar e consultar: em breve)                             |
| `/tarefas/nova/:caoId`                       | Nova tarefa: pontual ou programada, calendário, etapas do catálogo e responsáveis               |
| `/perfil`, `/notificacoes`, `/esqueci-senha` | Placeholders (próximas entregas)                                                                |

## Pré-requisitos

- Node.js 24.15+ (veja `.nvmrc`) e npm 11+
- Docker Desktop (para o PostgreSQL)

## Como rodar

```bash
# 1. Instalar dependências (raiz do monorepo)
npm install

# 2. Subir o banco (Postgres + Adminer)
npm run db:up            # ou: cd infra && docker compose up -d

# 3. Configurar e preparar a API
cp apps/api/.env.example apps/api/.env
npm run db:migrate -w @neurodog/api    # cria as tabelas
npm run db:seed -w @neurodog/api       # dados de exemplo (apaga e recria tudo)

# 4. Rodar a API → http://localhost:3000/api/v1/health
npm run dev:api

# 5. Rodar o app (em outro terminal) → http://localhost:4200
npm run dev:web
```

**Testar no celular:** com o celular no mesmo Wi-Fi do computador, rode `npm run dev:web:rede` e abra
`http://<IP-do-computador>:4200` no celular (o IP aparece em `ipconfig`, campo "Endereço IPv4"). Na
primeira vez o Windows pode pedir para liberar o Node no firewall: permita em "Redes privadas".

**Usuários do seed** (senha `neurodog123`): `arthur` (ADMIN), `camila`, `rafael`, `bruna`.

Outros comandos da API: `db:reset` (recria o banco do zero), `db:studio` (Prisma Studio),
`test`, `build`. Para ver o banco no navegador use o Adminer: http://localhost:8081.

## API (`/api/v1`)

Todas as rotas, exceto `health` e `auth`, exigem `Authorization: Bearer <accessToken>`.
Sucesso: `{ data, meta? }` · Erro: `{ error: { code, message, details? } }`.

| Método | Rota                               | O que faz                                                                                                   |
| ------ | ---------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| GET    | `/health`                          | API e banco no ar                                                                                           |
| POST   | `/auth/login`                      | Login (refresh token vai em cookie httpOnly)                                                                |
| POST   | `/auth/refresh` · `/auth/logout`   | Renova / encerra a sessão                                                                                   |
| GET    | `/auth/me`                         | Usuário logado                                                                                              |
| GET    | `/painel`                          | Números da home do usuário logado                                                                           |
| GET    | `/usuarios`                        | Usuários ativos (chips e badges)                                                                            |
| GET    | `/catalogo/categorias`             | Categorias de atividade com suas etapas                                                                     |
| GET    | `/caes`                            | Cães internados. Query: `busca`, `filtro` (TODOS, ATENCAO, MEDICACAO, SAINDO), `responsavelId`, `meus=true` |
| GET    | `/caes/:id`                        | Perfil do cão (dados da entrada, progresso, alertas)                                                        |
| POST   | `/caes`                            | Entrada de cão novo (multipart: `dados` JSON + `fotoFrente`, `fotoLateral`, `fotosDetalhe`)                 |
| GET    | `/caes/:id/rotina?data=AAAA-MM-DD` | Checklist do dia (padrão: hoje)                                                                             |
| POST   | `/caes/:id/tarefas`                | Cria tarefa (pontual ou programada) com etapas e responsáveis                                               |
| PATCH  | `/execucoes/:id`                   | Marca/desmarca item do checklist `{ concluida }`                                                            |
| GET    | `/caes/:id/ocorrencias`            | Histórico de ocorrências                                                                                    |
| POST   | `/caes/:id/ocorrencias`            | Registra ocorrência `{ descricao }`                                                                         |

## Estrutura

| Pasta                | Conteúdo                                                            |
| -------------------- | ------------------------------------------------------------------- |
| `apps/api`           | API REST (Express + TypeScript + Prisma)                            |
| `apps/web`           | App Angular (PWA)                                                   |
| `packages/contracts` | Schemas e tipos compartilhados entre API e web                      |
| `infra/`             | Docker Compose e infraestrutura local ([detalhes](infra/README.md)) |
| `docs/design/`       | Prints do Figma e logo                                              |

## Scripts da raiz

| Script                                  | O que faz                             |
| --------------------------------------- | ------------------------------------- |
| `npm run db:up` / `db:down` / `db:logs` | Controla o banco local                |
| `npm run lint` / `lint:fix`             | ESLint                                |
| `npm run format` / `format:check`       | Prettier                              |
| `npm run typecheck`                     | Checagem de tipos em todos os pacotes |
| `npm test`                              | Testes de todos os pacotes            |

## Instalar no celular (PWA)

O service worker só é ativado no build de produção (`npm run build -w @neurodog/web`, saída em
`apps/web/dist/web/browser`). Servido por HTTPS, o navegador oferece "Adicionar à tela inicial".

## Problemas comuns

| Sintoma                                         | Solução                                                                                     |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `docker: comando não encontrado`                | Abra um terminal novo após instalar o Docker Desktop e deixe o Docker Desktop aberto        |
| API não sobe: "Variáveis de ambiente inválidas" | Faltou `cp apps/api/.env.example apps/api/.env`                                             |
| `ng` reclama da versão do Node                  | Use Node 24.15+ (`node -v`)                                                                 |
| `npm install` avisa "install scripts"           | Pacotes liberados ficam em `allowScripts` no `package.json` (`npm install-scripts approve`) |
| Tela de login volta sempre                      | A API precisa estar rodando (`npm run dev:api`); o front usa o proxy `/api` → `:3000`       |
| Fotos de cães                                   | Ficam em `apps/api/uploads/` (fora do git). Para nuvem, troque o `StorageService`           |

