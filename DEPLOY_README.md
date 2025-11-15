# Guias de Deploy do OpenSheets

Escolha o guia apropriado para seu caso:

## 📚 Guias Disponíveis

### 1. [DEPLOY_COOLIFY.md](./DEPLOY_COOLIFY.md)
**Deploy no Coolify usando Docker**

**Use para:** Deploy em produção no Coolify

**Opções disponíveis:**
- ⚡ Com imagem do Docker Hub (deploy rápido)
- 🐢 Com build direto no Coolify (mais simples)
- 🔐 Com PostgreSQL separado (melhor gestão)

---

### 2. [DOCKER_HUB.md](./DOCKER_HUB.md)
**Publicação de imagens no Docker Hub**

**Use para:**
- Configurar CI/CD com GitHub Actions
- Publicar imagens automaticamente
- Deploy ultra-rápido (segundos vs minutos)

**Inclui:**
- Workflow GitHub Actions pronto
- Publicação automática e manual
- Gestão de versões e tags

---

## 🚀 Fluxo Recomendado para Produção

```
1. Configurar Docker Hub
   └─> Seguir: DOCKER_HUB.md
        ├─ Criar access token no Docker Hub
        ├─ Configurar secrets no GitHub
        └─ Push código → imagem publicada automaticamente

2. Deploy no Coolify
   └─> Seguir: DEPLOY_COOLIFY.md → Opção 1
        ├─ Usar docker-compose.prod.yml
        ├─ Configurar variáveis de ambiente
        └─ Deploy em segundos ⚡
```

---

## ⚡ Quick Start (Mais Rápido)

**Não quer configurar Docker Hub agora?**

```
└─> Seguir: DEPLOY_COOLIFY.md → Opção 2
     ├─ Usar docker-compose.yml original
     ├─ Configurar variáveis básicas
     └─ Deploy em 2-5 minutos 🐢
```

---

## 📁 Arquivos de Configuração

| Arquivo | Uso |
|---------|-----|
| `docker-compose.yml` | Desenvolvimento local + Deploy simples no Coolify |
| `docker-compose.prod.yml` | Produção com imagem do Docker Hub |
| `docker-compose.coolify.yml` | Deploy no Coolify com PostgreSQL separado |
| `.env.production.example` | Template de variáveis de ambiente |
| `.github/workflows/docker-publish.yml` | CI/CD automático para Docker Hub |

---

## 🔧 Variáveis de Ambiente

Ver template completo: [.env.production.example](./.env.production.example)

### Mínimo necessário:
```bash
# PostgreSQL
POSTGRES_PASSWORD=...
DATABASE_URL=postgresql://opensheets:senha@db:5432/opensheets_db

# Better Auth
BETTER_AUTH_SECRET=...
BETTER_AUTH_URL=https://seu-dominio.com
```

### Gerar secrets:
```bash
openssl rand -base64 32
```

---

## 🆘 Troubleshooting

| Problema | Solução |
|----------|---------|
| Deploy muito lento | Use Docker Hub (DOCKER_HUB.md) |
| Erro de conexão com banco | Verificar DATABASE_URL |
| Imagem não encontrada | Configurar DOCKER_USERNAME no Coolify |
| Migrations não executam | Executar manualmente: `pnpm db:push` |

---

## 📖 Mais Recursos

- [Documentação Coolify](https://coolify.io/docs)
- [Docker Hub](https://hub.docker.com/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Next.js Deployment](https://nextjs.org/docs/deployment)

---

## 🎯 Resumo

**Para produção (recomendado):**
1. DOCKER_HUB.md → Configurar CI/CD
2. DEPLOY_COOLIFY.md → Opção 1 (Docker Hub)

**Para começar rápido:**
- DEPLOY_COOLIFY.md → Opção 2 (Build direto)

**Dúvidas?**
- Abra uma issue no GitHub
- Consulte os guias detalhados acima
