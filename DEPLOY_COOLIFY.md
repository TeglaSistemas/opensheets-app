# Deploy do OpenSheets no Coolify

Este guia mostra como fazer deploy do OpenSheets no Coolify usando Docker.

## 🚀 Resumo Rápido

**Duas formas principais:**

### 1. Com imagem do Docker Hub (MAIS RÁPIDO ⚡)
- Publicar imagem no Docker Hub primeiro
- Deploy em segundos (só faz pull)
- Ideal para produção e CI/CD
- **Ver guia:** [DOCKER_HUB.md](./DOCKER_HUB.md)

### 2. Build direto no Coolify (MAIS SIMPLES)
- Usar `docker-compose.yml` original
- Coolify faz build completo (2-5 min)
- Ideal para começar rápido

---

## Qual opção escolher?

| Opção | Quando usar | Deploy | Complexidade |
|-------|-------------|--------|--------------|
| **Opção 1: Imagem Docker Hub** | Deploy rápido, CI/CD, produção | ⚡ Segundos | ⭐ Fácil |
| **Opção 2: Build no Coolify** | Começar rápido, teste | 🐢 2-5 min | ⭐ Fácil |
| **Opção 3: PostgreSQL separado** | Backups automáticos, melhor gestão | 🐢 2-5 min | ⭐⭐ Médio |

**Recomendado para produção: Opção 1 (Docker Hub)** ✅

---

## Opção 1: Deploy com Docker Hub (Recomendado para Produção)

**Vantagens:**
- ✅ Deploy em **segundos** (vs minutos)
- ✅ Mesma imagem em múltiplos ambientes
- ✅ CI/CD automático via GitHub Actions
- ✅ Rollback fácil (trocar tag)

### Passo 0: Publicar Imagem no Docker Hub

**Siga o guia completo:** [DOCKER_HUB.md](./DOCKER_HUB.md)

Resumo rápido:
1. Configurar secrets no GitHub (DOCKER_USERNAME e DOCKER_PASSWORD)
2. Push para main ou criar tag → GitHub Actions publica automaticamente
3. Verificar imagem em https://hub.docker.com/

### Passo 1: Criar Aplicação no Coolify

1. No Coolify: **Projects** → **+ New Resource** → **Docker Compose Empty**
2. Configure:
   - **Name**: `opensheets`
   - **Source**: Conecte ao seu repositório Git
   - **Branch**: `main`
   - **Docker Compose Location**: `docker-compose.prod.yml`

### Passo 2: Configurar Variáveis de Ambiente

```bash
# Configuração da imagem do Docker Hub
DOCKER_USERNAME=seu-usuario-dockerhub
IMAGE_TAG=latest

# PostgreSQL
POSTGRES_USER=opensheets
POSTGRES_PASSWORD=SENHA_FORTE_AQUI
POSTGRES_DB=opensheets_db
DATABASE_URL=postgresql://opensheets:SENHA_FORTE_AQUI@db:5432/opensheets_db

# Better Auth (OBRIGATÓRIO)
BETTER_AUTH_SECRET=GERAR_COM_OPENSSL
BETTER_AUTH_URL=https://seu-dominio.com

# Opcionais: EMAIL, OAUTH, AI (ver abaixo)
```

### Passo 3: Deploy

1. Clique em **Deploy**
2. **Coolify vai fazer PULL da imagem** (segundos) ⚡
3. PostgreSQL sobe primeiro
4. App executa migrations e inicia

---

## Opção 2: Deploy com Build no Coolify (Mais Simples)

**Use se:** Quer começar rápido sem configurar Docker Hub

### Passo 1: Criar Aplicação no Coolify

1. **Projects** → **+ New Resource** → **Docker Compose Empty**
2. Configure:
   - **Name**: `opensheets`
   - **Source**: Conecte ao repositório
   - **Branch**: `main`
   - **Docker Compose Location**: `docker-compose.yml` (ou deixe vazio)

### Passo 2: Variáveis de Ambiente

```bash
# PostgreSQL
POSTGRES_USER=opensheets
POSTGRES_PASSWORD=SENHA_FORTE_AQUI
POSTGRES_DB=opensheets_db
DATABASE_URL=postgresql://opensheets:MESMA_SENHA_ACIMA@db:5432/opensheets_db

# Better Auth
BETTER_AUTH_SECRET=GERAR_COM_OPENSSL
BETTER_AUTH_URL=https://seu-dominio.com
```

**Gerar secrets:**
```bash
openssl rand -base64 32  # Para POSTGRES_PASSWORD
openssl rand -base64 32  # Para BETTER_AUTH_SECRET
```

**⚠️ IMPORTANTE:** A senha em `POSTGRES_PASSWORD` e `DATABASE_URL` deve ser igual!

### Passo 3: Deploy

1. Clique em **Deploy**
2. Coolify faz **build completo** (2-5 min) 🐢
3. PostgreSQL e app sobem juntos

---

## Opção 3: PostgreSQL Separado

**Vantagens:**
- ✅ Backups automáticos do Coolify
- ✅ Monitoring independente
- ✅ Melhor para produção

<details>
<summary>Ver instruções detalhadas</summary>

### Passo 1: Criar PostgreSQL

1. **Databases** → **+ New Database** → **PostgreSQL 18**
2. Configure nome, usuário, senha
3. Copie a **Internal Connection String**

### Passo 2: Criar Aplicação

1. **Docker Compose Empty** → apontar para `docker-compose.coolify.yml`
2. Configurar variáveis (DATABASE_URL aponta para PostgreSQL separado)

</details>

---

## Variáveis de Ambiente Opcionais

```bash
# Email (Resend)
RESEND_API_KEY=re_...
EMAIL_FROM=noreply@seu-dominio.com

# OAuth Google
GOOGLE_CLIENT_ID=...
GOOGLE_CLIENT_SECRET=...

# OAuth GitHub
GITHUB_CLIENT_ID=...
GITHUB_CLIENT_SECRET=...

# AI Providers
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
GOOGLE_GENERATIVE_AI_API_KEY=...
OPENROUTER_API_KEY=...
```

---

## Configurar Domínio

1. No Coolify: **Domains**
2. Adicione seu domínio
3. SSL automático via Let's Encrypt

---

## Verificação de Deploy

1. **Health Check**: `https://seu-dominio.com/api/health`
   ```json
   {"status":"ok","timestamp":"...","service":"opensheets-app"}
   ```

2. **Logs**: No Coolify → Logs

3. **Database**: Verificar se migrations rodaram

---

## Comparação das Opções

| | Docker Hub | Build no Coolify | PostgreSQL Separado |
|---|---|---|---|
| **Tempo de deploy** | ⚡ Segundos | 🐢 2-5 min | 🐢 2-5 min |
| **Complexidade** | Médio (setup inicial) | Fácil | Médio |
| **CI/CD** | ✅ Automático | ❌ Manual | ✅ Possível |
| **Rollback** | ✅ Fácil (trocar tag) | ❌ Rebuild | ✅ Fácil |
| **Backups DB** | ❌ Manual | ❌ Manual | ✅ Automático |
| **Ideal para** | Produção | Desenvolvimento/Teste | Produção crítica |

---

## Atualizações

### Com Docker Hub (Opção 1):
```bash
# Fazer alterações
git add .
git commit -m "feat: nova funcionalidade"
git push origin main
# → GitHub Actions publica nova imagem automaticamente

# No Coolify: Redeploy (faz pull da nova imagem)
```

### Com Build no Coolify (Opção 2):
```bash
git push origin main
# No Coolify: Redeploy (faz build novamente)
```

---

## Troubleshooting

### Deploy demora muito
**Causa:** Fazendo build completo
**Solução:** Use Opção 1 (Docker Hub)

### Erro de conexão com banco
**Causa:** DATABASE_URL incorreta
**Solução:** Verificar host (`db` para compose local, ou nome do serviço no Coolify)

### Migrations não executam
**Solução:** Executar manualmente no terminal do container:
```bash
pnpm db:push
```

---

## Recursos

- [Documentação Coolify](https://coolify.io/docs)
- [Guia Docker Hub](./DOCKER_HUB.md)
- [Next.js Deployment](https://nextjs.org/docs/deployment)
- [Drizzle Migrations](https://orm.drizzle.team/docs/migrations)

---

## Próximos Passos

1. ✅ Configurar domínio personalizado
2. ✅ Configurar OAuth (Google, GitHub)
3. ✅ Configurar email (Resend)
4. ✅ Configurar AI providers
5. ✅ Configurar backups (se PostgreSQL separado)
6. ✅ Configurar monitoring
