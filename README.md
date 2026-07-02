# 🎮 ESPORT ARENA

### Plataforma SaaS para Criação e Gestão de Ligas de FIFA/EA FC

---

## 📌 Visão Geral

**ESPORT ARENA** é uma plataforma completa para organizadores de ligas de e-sports (FIFA/EA FC). O sistema automatiza todo o ciclo de vida de uma liga: criação de campeonatos, gestão de times e jogadores, draft ao vivo, geração de partidas, inserção de resultados e classificação em tempo real.

O projeto foi desenvolvido para resolver um problema real: **organizadores de ligas amadoras perdem horas montando squads manualmente, controlando resultados em planilhas e atualizando tabelas no braço.** Com o ESPORT ARENA, tudo é feito em poucos cliques.

> **Status atual:** MVP funcional · Sprints 1 a 6 entregues · Próximo: painel CEO (Sprint 7) · Testes automatizados (Pest) rodando

---

## 🚀 Funcionalidades Principais

### 🏆 Gestão de Campeonatos
- CRUD completo de campeonatos
- Suporte a **Pontos Corridos (com turno e returno)** e **Mata-mata (com árvore completa)**
- Configuração de: nome, formato, número de times, sistema de draft, taxa de inscrição, prize pool, visibilidade (público/privado)
- Status automático: `inscrições → draft → andamento → encerrado`
- Botão manual para gerar partidas (com validação de duplicidade)

### 🧑‍🤝‍🧑 Gestão de Times e Jogadores
- Admin pode cadastrar times e associar a usuários (jogadores)
- Validação: não exceder `max_teams` e um usuário não pode ter dois times no mesmo campeonato
- Orçamento inicial padrão (R$ 50.000.000) e estatísticas zeradas
- Admin cria usuários com nome, e-mail, senha e papel (admin/player)

### ⚡ Draft Serpentine Ao Vivo
- Ordem serpentine: `1,2,...,N,N,...,2,1...`
- **10 rodadas fixas** por time (total picks = times × 10)
- **Timer de 30 segundos** por turno (cache + polling a cada 1 segundo)
- Jogadores elegíveis: top 500 com vínculo a clube (`teams_globals.type = 'club'`)
- Ações: Escolher, Pular, Pausar/Retomar
- Proteção anti‑clique duplo
- Finalização automática com geração de partidas

### 📅 Geração de Partidas
- Suporte a **Pontos Corridos (turno e returno)** e **Mata-mata**
- Bloqueio de duplicidade (se já existirem partidas, retorna erro 422)
- Atualização do campeonato: `total_rounds` e `current_round`

### ⚽ Gestão de Partidas e Resultados
- Listagem com abas: Pendentes, Validadas, Divergência
- Inserção de resultados via modal (`home_score`, `away_score`)
- Serviço `StandingsServices` recalcula classificação automaticamente
- Exibição de placar nas partidas validadas

### 📊 Página de Detalhes do Campeonato
- Classificação real (posição, jogos, pontos, gols, forma)
- Gráfico de gols por rodada (barras)
- Estatísticas gerais (total de gols, média, distribuição de resultados)
- Aba de artilheiros (top 10)
- Permissão por tenant

### 👥 Listagem de Jogadores
- Filtros por nome, posição, overall mínimo
- Ordenação por coluna (clique no cabeçalho)
- Paginação (16 itens/página)
- Modal detalhado com atributos técnicos, clube/seleção

### 📦 Importação de Dados
- Importação de **20.000+ jogadores FIFA** via CSV
- Importação de clubes/seleções (`teams_globals`)
- Importação de vínculos jogador ↔ time global

### 🔐 Autenticação e Multitenancy
- Login/registro com Laravel Breeze
- Isolamento total por `tenant_id` (admin só vê seus próprios dados)
- Middleware de permissão com hierarquia: `ceo > admin > player`
- Rotas protegidas por papel (`role:admin`, `role:player`, `role:ceo`)

### 🛒 Mercado de Transferências (Sprint 6)
- Janelas de mercado multi-campeonato
- Faixas de preço por overall (configurável pelo admin)
- Leilão com **anti-snipe** (últimos 10 segundos estendem o leilão)
- Compra direta (Buy Now) por valor fixo
- Venda de jogadores entre jogadores
- Comando scheduler `market:process-auctions` (roda a cada 30s)
- Free agents (jogadores sem time) auto-populados via query

### 🎮 Squad Generator (🚧 Em desenvolvimento)
- Interface frontend já implementada (página `/admin/squad` com histórico e simulação)
- Serviço C# funcional (já gera o arquivo `.squad` a partir dos picks)
- **Pendente:** integração completa com o Laravel via fila (Job) para processamento assíncrono e download real
- Previsão: após a Sprint 7 (CEO)

### 🧹 Dívida Técnica Resolvida
- Correção de N+1 queries
- Transações em operações críticas (`DB::transaction`)
- Soft deletes em `teams` e `users`
- Auditoria de edição de resultados (log em `notifications_logs`)
- Cálculo de maior goleada (`calculateHighlights`)
- Comando Artisan para corrigir rounds de campeonatos antigos (`championship:fix-rounds`)
- Extração de lógica duplicada para `StandingsServices::syncRound()`

---

## 🛠️ Stack Tecnológica

| Camada | Tecnologia |
|--------|------------|
| **Backend** | PHP 8.4+, Laravel 11, Eloquent ORM, Cache (file/redis) |
| **Frontend** | Vue 3 (Composition API), Inertia.js, Vite, CSS customizado |
| **Banco de Dados** | MySQL 8.0 / MariaDB |
| **Autenticação** | Laravel Breeze (sessões) |
| **Multitenancy** | Isolamento por `tenant_id` em todas as tabelas principais |
| **Testes** | Pest (unitários + feature) |
| **Gerenciamento de filas** | Redis + Laravel Horizon (planejado) |
| **Notificações (futuro)** | Z-API / Evolution API (WhatsApp), Mailgun/SES (email) |
| **Pagamentos (futuro)** | Stripe / Asaas |

---

## 🧠 Regras de Negócio

| Área | Regra |
|------|-------|
| **Tenant** | Cada admin tem seu próprio tenant; todos os recursos são isolados por `tenant_id`. |
| **Campeonato** | `max_teams` entre 2 e 32. Status: `inscricoes` → `draft` → `andamento` → `encerrado`. |
| **Time** | Um usuário só pode ter um time por campeonato. Número de times ≤ `max_teams`. |
| **Draft** | Apenas jogadores com clube (`teams_globals.type = 'club'`) e top 500 por overall. 10 rodadas fixas. Timer de 30s por turno. |
| **Partida** | Somente admin insere resultados (MVP). Ao validar, classificação é recalculada. |
| **Classificação** | Calculada a partir de partidas validadas: pontos (V=3, E=1, D=0), saldo de gols, gols marcados. |
| **Mercado** | Free agents listados automaticamente. Lances com retenção de saldo e anti-snipe. Compra imediata transfere jogador e moedas. |

---

## 🧪 Comandos Úteis

```bash
# Importar jogadores FIFA (CSV)
php artisan players:import storage/app/players.csv --chunk=1000

# Importar clubes/seleções
php artisan import:teams-global storage/app/times.csv

# Importar vínculos jogador ↔ clube
php artisan import:team-player-links storage/app/links.csv

# Limpar cache do draft (ex: campeonato ID 1)
php artisan tinker
>>> Cache::forget('draft:1');

# Corrigir rounds de campeonatos antigos
php artisan championship:fix-rounds --id=1

# Processar leilões do mercado
php artisan market:process-auctions

# Link simbólico para imagens dos jogadores
php artisan storage:link

# Limpar cache geral
php artisan optimize:clear

# Rodar testes
php artisan test
🗺️ Roadmap
Sprint 1 – Correções e dívida técnica (N+1, transações, typos)

Sprint 2 – Middleware de role + permissões

Sprint 3 – Polimentos do MVP (current_round, artilheiros, CRUD, soft deletes)

Sprint 4 – Páginas do jogador com dados reais

Sprint 5 – Dashboard admin com estatísticas e gráficos

Sprint 6 – Mercado de transferências (leilão, compra, anti-snipe)

Sprint 7 – Páginas CEO (gestão de tenants, relatórios, logs)

**Pós-MVP** – Squad Generator (integração completa com serviço C#)

Sprint 8 – Time estável entre temporadas (refatoração estrutural)

📄 Licença
MIT

📬 Contato
LinkedIn: https://www.linkedin.com/in/alexander-hartmann17/

E-mail: alexalveshart@gmail.com

⭐ Se você gostou do projeto, deixe uma estrela no repositório!