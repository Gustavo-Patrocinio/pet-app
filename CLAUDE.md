# Adotaí — app de adoção de animais

Marketplace de adoção no estilo classificados (não swipe). ONGs anunciam cães e gatos, adotantes
buscam por proximidade e pela qualidade da ONG, conversam por chat, e o pós-adoção acontece na
plataforma. Mobile (adotante) + web (adotante, ONG e admin).

- Plano completo (doc vivo): https://claude.ai/artifact/JYvjmQTs474FCXi6pCnGjB
- Protótipo de telas (canvas): https://claude.ai/artifact/G6SqL67ZWZ4opptb86oM4s

## Decisões já tomadas (não reabrir sem o dono pedir)

| Tema | Decisão |
| --- | --- |
| Formato | Classificados: lista + mapa + filtros. Nada de swipe/"Tinder" |
| Espécies | Só cães e gatos |
| Quem anuncia | Só ONGs com CNPJ no MVP. Protetores independentes ficam para depois |
| Cidade piloto | Ribeirão Preto (SP) |
| Nome | Adotaí (provisório) |
| Termo de adoção | Modelo padrão da plataforma (ONG não sobe o próprio por enquanto) |
| Receita | MVP 100% gratuito. Depois testar patrocínio local (pet shops/clínicas). Ainda em aberto |
| LGPD | Obrigatório desde o MVP: política de privacidade, consentimento, retenção, exclusão de conta |
| Backend | Supabase (Postgres + PostGIS, Auth, Realtime, Storage, Edge Functions) |
| Mobile | React Native + Expo (Expo Router) |
| Web | Next.js (App Router) — páginas públicas indexáveis (SEO) + painéis |

## Regras de produto que o código DEVE respeitar

1. **Cadastro em duas etapas.**
   - Etapa 1 (criar conta): nome completo, e-mail, senha (ou Google/Apple). Libera: buscar, ver,
     favoritar, buscas salvas, conversar com ONG.
   - Etapa 2 (completar perfil): celular (verificado por SMS), CPF (validado), CEP → preenche
     cidade/UF/bairro via ViaCEP, endereço (rua e número) e complemento digitados, perfil de moradia.
     Só com a etapa 2 completa o botão **"Quero adotar"** envia pedido. Se incompleto, abre a tela de
     completar perfil e depois volta direto ao pedido daquele animal.
2. **Privacidade do adotante.** A ONG NUNCA vê o número do CPF — só "CPF verificado". Celular e
   endereço completo só ficam visíveis para a ONG depois que ela **aprova** o pedido. Antes disso,
   só bairro + distância. CPF único por conta.
3. **Chat sempre ligado a um animal.** Quem inicia é o adotante. ONG não inicia conversa com quem não
   demonstrou interesse. Chat tem card do animal + status do pedido no topo, proposta de visita,
   denunciar/bloquear. A mesma conversa continua no pós-adoção.
4. **Ranking da ONG (0–5)**, calculado pela plataforma (não só estrelas):
   sucesso 90 dias 30% · pós-adoção cumprido 20% · avaliações 20% · tempo de resposta 15% ·
   transparência 15%. ONG com < 5 adoções concluídas mostra "Nova na plataforma". Avaliação só de
   quem adotou pelo app, uma por adoção. Ordenação "recomendado" = 60% proximidade + 40% nota.
5. **Pós-adoção:** check-ins em 7, 30, 90, 180 dias (MVP: 7 e 30). Alerta para a ONG se atrasar ou
   se o adotante tocar "estou com dificuldade".
6. **Endereços:** guardar ponto exato (PostGIS), exibir só bairro e distância arredondada.

## Arquitetura do repositório

```
app-pet/
  apps/
    web/        Next.js — site público (busca, animal, ONG), área do adotante, painel ONG, admin
    mobile/     Expo — app do adotante
  packages/
    shared/     TypeScript puro: tipos do banco, validações (zod), CPF, CEP/ViaCEP, constantes
  supabase/
    migrations/ SQL versionado (schema, RLS, funções de busca e ranking)
    seed.sql    Dados fictícios de Ribeirão Preto para desenvolvimento
  CLAUDE.md
```

- npm workspaces (pnpm não está instalado na máquina).
- Código, nomes de tabelas e colunas em **inglês**; textos de interface em **português (pt-BR)**.
- Lógica de negócio importante (busca por raio, ranking, regras de visibilidade) fica no **Postgres**
  (funções SQL + RLS), para não depender de recursos exclusivos do Supabase e facilitar migração.
- Nunca commitar `.env` / chaves. Usar `.env.example` com placeholders.

## Modelo de dados (resumo)

`profiles` (1:1 auth.users, role adopter|ngo_member|admin, dados da etapa 2) ·
`ngos` (CNPJ, verified, location geography, rating) · `ngo_members` ·
`animals` (species dog|cat, size, age, sex, saúde, compatibilidades, status, location) ·
`animal_photos` · `adoption_requests` (status: sent→reviewing→visit→approved|rejected) ·
`conversations` (adopter + ngo + animal) · `messages` · `adoptions` · `checkins` · `reviews` ·
`favorites` · `saved_searches`.

## Design (do protótipo)

Fonte Plus Jakarta Sans. Cores: primária `#0F6E5C` (hover `#0A4F42`), destaque `#E8833A` (texto
escuro sobre ele), fundo `#F6F7F5`, texto `#1B1F1D`, texto secundário `#5B635F`, borda `#E4E7E3`,
chip suave `#E6F2EE`. Cantos 12–16px. Alvos de toque ≥ 44px. Ícones de traço (sem emoji).

## Plano de ação

Legenda: [x] feito · [~] em andamento · [ ] a fazer · 🔒 precisa do dono

### Fase 0 — Fundação
- [x] Plano e protótipo
- [x] Monorepo (npm workspaces: apps/web + packages/*), git, .gitignore, .env.example
- [x] `packages/shared`: tipos gerados do banco, CPF/CNPJ, máscaras, CEP (BrasilAPI→ViaCEP→Nominatim), rótulos pt-BR, testes
- [x] Migrations: schema, PostGIS, índices, RLS, funções (busca por raio, pedidos, contato pós-aprovação, ranking, LGPD)
- [x] Seed com dados fictícios de Ribeirão Preto (usuários de teste, senha `adotai123`)
- [x] Testes de RLS/privacidade rodados no banco local (ver log)
- [x] Projeto Supabase de produção criado pelo dono: ref `noeuygatrerbsbcnhpek`, região São Paulo (sa-east-1).
      Migrations aplicadas (sem seed). Envs: `.env` (ref + senha, CLI), `apps/web/.env.production.local`, `apps/mobile/.env.production`.
      Push sem login: `npx supabase db push --db-url postgresql://postgres.<ref>:<senha-urlencoded>@aws-0-sa-east-1.pooler.supabase.com:5432/postgres`

### Fase 1 — MVP (~3 meses)
- [x] Web: busca (lista + mapa MapLibre/OpenFreeMap + filtros + ordenação + paginação), página do animal, página da ONG, lista de ONGs
- [x] Web: cadastro/login (etapa 1) e completar perfil (etapa 2) com SMS e CEP
- [x] Web: painel da ONG — visão geral, animais (criar/editar/fotos), candidatos, aprovar/recusar, confirmar adoção, pós-adoção, dados da ONG
- [x] Web: chat em tempo real (respostas rápidas, proposta de visita, confirmação)
- [x] Web: meus pedidos (cancelar, responder check-in), favoritos, perfil, exclusão de conta
- [x] Web: admin (verificar ONG, denúncias), cadastro de ONG (/sou-ong), privacidade e termo (rascunhos)
- [x] Mobile (v1): busca (filtros, localização), animal, favoritos, chat realtime, perfil + pedidos, login, cadastro, completar perfil
      (tsc + expo lint ok; `expo export --platform android` compila). **Falta testar em aparelho** (Expo Go).
- [ ] Mobile: mapa (MapLibre RN exige development build), push, responder check-in, propor/confirmar visita já funciona
- [ ] Problema conhecido: no alvo **web** do Expo as abas quebram ("array of styles to a child of <Slot>",
      expo-router SDK 58). Por isso `platforms: ["ios","android"]` no app.json; a web é o Next.js.
- [ ] `pg_cron` chamando `run_daily_jobs()` (check-ins atrasados + recálculo de notas) — no projeto de produção
- [ ] Notificações: push (Expo) e e-mail (mensagem nova, check-in, pedido)
- [ ] Denunciar/bloquear na interface do chat (tabela `reports` e coluna `blocked_at` já existem)
- [ ] Testes E2E (Playwright) dos fluxos principais
- [ ] 🔒 Revisão jurídica da Política de Privacidade e do Termo de adoção (rascunhos em /privacidade e /termo-de-adocao)
- [ ] 🔒 Provedor de SMS em produção (Twilio, MessageBird ou Vonage — configurar no Supabase Auth)
- [x] ~~Chave de mapas~~ — decidido: mapa gratuito (MapLibre + OpenFreeMap), sem chave
- [ ] Critério para avançar: 5 ONGs ativas em Ribeirão Preto e 30 adoções registradas

### Fase 2
Ranking + avaliações, buscas salvas com alerta, termo digital, agendamento de visita no chat,
check-ins 90/180, carteira de saúde com lembretes.

### Fase 3
Relatórios para ONGs, doações via Pix, equipe com papéis, moderação automática, app da ONG.

## Comandos

```bash
npm install                # raiz (web + shared + CLI supabase)
npx supabase start         # banco local no Docker (aplica migrations + seed)
npm run db:reset           # recria o banco local
npm run db:types           # regenera packages/shared/src/database.types.ts — rodar após mudar migrations
npm run dev:web            # http://localhost:3000
npm run build:web && npm run lint:web
node --test packages/shared/src/shared.test.ts
node scripts/screenshots.mjs <animal-id>   # imagens do README (site rodando em :3000)
```

- SMS local é simulado (supabase/config.toml → `[auth.sms.test_otp]`): celular (16) 99999-0000, código 123456.
- **Ambiente Windows:** o fnm quebra quando o comando faz `cd` para subpastas. Rode tudo a partir da raiz
  (`npm -w web …`, `npx --prefix apps/web …`). No Git Bash, `docker exec` com caminhos precisa de `MSYS_NO_PATHCONV=1`.
- `apps/mobile` NÃO está nos workspaces (React 19.3 do Expo SDK 58 ≠ React 19.1 do Next 15). Instale
  dependências dentro dele com `npx expo install`. Leia `apps/mobile/AGENTS.md` (Expo muda muito entre SDKs).
- Tailwind v4: classes próprias (btn, field, card…) ficam em `@utility` no globals.css, não em `@layer components`.

## O que depende do dono (🔒)

1. ~~Projeto Supabase~~ feito. Falta: configurar Auth no painel (Site URL / redirect URLs do domínio, confirmação de e-mail).
2. Escolher provedor de SMS para produção (é pago por mensagem).
3. Revisão jurídica: Política de Privacidade, Termo de adoção, e-mail do encarregado de dados (LGPD).
4. Deploy da web (sugestão: Vercel, plano gratuito) — precisa da conta dele.
5. Contas de loja (Google Play US$ 25 único; Apple US$ 99/ano) quando o app mobile estiver pronto.
6. Modelo de receita (em aberto; MVP gratuito).

## Log de progresso

- 2026-11-18: plano, protótipo e decisões fechadas com o dono. Início da Fase 0.
- 2026-11-18 (madrugada, trabalho autônomo): Fase 0 completa; web do MVP completa e compilando
  (23 rotas, tsc + eslint + next build ok). Testes no banco local: busca por raio ok; adotante sem perfil
  completo recebe PROFILE_INCOMPLETE; ONG só recebe contato após aprovar; ONG2 não vê pedidos da ONG1;
  usuário não consegue trocar o próprio papel. README com capturas geradas por scripts/screenshots.mjs.
  Repositório: https://github.com/Gustavo-Patrocinio/pet-app (push autorizado pelo dono).
- 2026-11-18: banco de produção no ar (4 migrations aplicadas, API testada com a publishable key).
