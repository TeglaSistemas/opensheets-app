# Publicação no Docker Hub

Este guia mostra como publicar a imagem do OpenSheets no Docker Hub e usar no Coolify.

## 🚀 Resumo Rápido

1. **Configurar secrets no GitHub** (DOCKER_USERNAME e DOCKER_PASSWORD)
2. **Push para main ou criar tag** → GitHub Actions publica automaticamente
3. **No Coolify:** usar `docker-compose.prod.yml` apontando para sua imagem do Docker Hub

---

## Opção 1: Publicação Automática (GitHub Actions)

### Passo 1: Criar Access Token no Docker Hub

1. Acesse [Docker Hub](https://hub.docker.com/)
2. Faça login na sua conta
3. Vá em **Account Settings** → **Security** → **New Access Token**
4. Configure:
   - **Description**: `GitHub Actions - OpenSheets`
   - **Access permissions**: `Read & Write`
5. Clique em **Generate** e **copie o token** (você não verá novamente!)

### Passo 2: Configurar Secrets no GitHub

1. Vá no seu repositório GitHub
2. **Settings** → **Secrets and variables** → **Actions**
3. Clique em **New repository secret**
4. Adicione 2 secrets:

**Secret 1:**
- **Name**: `DOCKER_USERNAME`
- **Value**: seu username do Docker Hub (ex: `felipecoutinho`)

**Secret 2:**
- **Name**: `DOCKER_PASSWORD`
- **Value**: o access token copiado acima

### Passo 3: Publicar a Imagem

O workflow `.github/workflows/docker-publish.yml` já está configurado e vai publicar automaticamente quando você:

#### Opção A: Push para main (cria tag `latest`)
```bash
git add .
git commit -m "feat: nova versão"
git push origin main
```

Isso vai gerar as tags:
- `seu-usuario/opensheets:latest`
- `seu-usuario/opensheets:main`
- `seu-usuario/opensheets:main-abc123` (hash do commit)

#### Opção B: Criar release com tag semântica (recomendado)
```bash
# Criar tag de versão
git tag v1.0.0
git push origin v1.0.0
```

Isso vai gerar as tags:
- `seu-usuario/opensheets:latest`
- `seu-usuario/opensheets:1.0.0`
- `seu-usuario/opensheets:1.0`
- `seu-usuario/opensheets:1`

### Passo 4: Verificar Publicação

1. Acesse [Docker Hub](https://hub.docker.com/)
2. Vá em **Repositories**
3. Você verá `seu-usuario/opensheets` com as tags publicadas

Ou verifique pelo terminal:
```bash
docker pull seu-usuario/opensheets:latest
```

---

## Opção 2: Publicação Manual

Se preferir publicar manualmente sem GitHub Actions:

### Passo 1: Login no Docker Hub
```bash
docker login
# Digite seu username e password/token
```

### Passo 2: Build da Imagem
```bash
# Build com tag do seu usuário
docker build -t seu-usuario/opensheets:latest .

# Opcional: criar tags adicionais
docker tag seu-usuario/opensheets:latest seu-usuario/opensheets:v1.0.0
```

### Passo 3: Push para o Docker Hub
```bash
# Enviar a imagem
docker push seu-usuario/opensheets:latest

# Se criou tags adicionais
docker push seu-usuario/opensheets:v1.0.0
```

### Passo 4: Verificar
```bash
docker pull seu-usuario/opensheets:latest
```

---

## Usando no Coolify

### Opção A: Docker Compose Production (Recomendado)

1. **No Coolify:**
   - Projects → + New Resource → **Docker Compose Empty**
   - Conecte ao repositório
   - **Docker Compose Location**: `docker-compose.prod.yml`

2. **Configure variáveis de ambiente:**
   ```bash
   # Configuração da imagem do Docker Hub
   DOCKER_USERNAME=seu-usuario
   IMAGE_TAG=latest

   # PostgreSQL
   POSTGRES_USER=opensheets
   POSTGRES_PASSWORD=SENHA_FORTE_AQUI
   POSTGRES_DB=opensheets_db
   DATABASE_URL=postgresql://opensheets:SENHA_FORTE_AQUI@db:5432/opensheets_db

   # Better Auth
   BETTER_AUTH_SECRET=GERAR_COM_OPENSSL
   BETTER_AUTH_URL=https://seu-dominio.com
   ```

3. **Deploy**
   - Clique em Deploy
   - O Coolify vai fazer **pull da imagem** do Docker Hub (não build!)
   - Muito mais rápido! ⚡

### Opção B: Editar docker-compose.prod.yml antes

Você pode editar o arquivo e fixar sua imagem:

```yaml
app:
  image: felipecoutinho/opensheets:latest  # Seu usuário aqui
  # ... resto da configuração
```

Depois fazer commit e push. No Coolify vai usar direto a imagem configurada.

---

## Vantagens de Usar Imagem do Docker Hub

✅ **Deploy mais rápido** - Apenas pull da imagem (segundos) vs build completo (minutos)
✅ **Mesma imagem em múltiplos ambientes** - Staging, produção, testes
✅ **Rollback fácil** - Trocar tag da imagem para versão anterior
✅ **CI/CD automatizado** - Push no GitHub → build automático → disponível no hub
✅ **Multi-arch** - Suporte para AMD64 e ARM64 (servidores diferentes)

---

## Gestão de Versões

### Estratégia Recomendada

```bash
# Desenvolvimento contínuo
main branch → seu-usuario/opensheets:latest

# Releases estáveis
v1.0.0 → seu-usuario/opensheets:1.0.0
v1.1.0 → seu-usuario/opensheets:1.1.0
v2.0.0 → seu-usuario/opensheets:2.0.0
```

### Exemplo de Workflow Completo

```bash
# 1. Desenvolver feature
git checkout -b feature/nova-funcionalidade
# ... fazer alterações ...
git add .
git commit -m "feat: adiciona nova funcionalidade"

# 2. Merge para main
git checkout main
git merge feature/nova-funcionalidade
git push origin main
# → GitHub Actions publica seu-usuario/opensheets:latest

# 3. Criar release quando estável
git tag v1.1.0
git push origin v1.1.0
# → GitHub Actions publica:
#   - seu-usuario/opensheets:1.1.0
#   - seu-usuario/opensheets:1.1
#   - seu-usuario/opensheets:1
#   - seu-usuario/opensheets:latest
```

### No Coolify: Controle de Versões

```bash
# Usar sempre a última versão (auto-update)
IMAGE_TAG=latest

# Fixar versão específica (produção estável)
IMAGE_TAG=1.0.0

# Usar versão minor mais recente
IMAGE_TAG=1.1
```

---

## Troubleshooting

### Erro: "unauthorized: authentication required"

**Causa:** Secrets não configurados ou token inválido

**Solução:**
1. Verifique se DOCKER_USERNAME e DOCKER_PASSWORD estão nos secrets do GitHub
2. Gere um novo access token no Docker Hub
3. Atualize o secret DOCKER_PASSWORD

### Erro: "denied: requested access to the resource is denied"

**Causa:** Username incorreto ou permissões insuficientes

**Solução:**
1. Verifique se DOCKER_USERNAME está correto (lowercase)
2. Certifique-se que o access token tem permissão "Read & Write"

### Build demora muito no GitHub Actions

**Causa:** Cache não está funcionando

**Solução:**
- O workflow já usa cache (buildx cache)
- Espere o primeiro build (demora mais)
- Builds seguintes serão mais rápidos

### Coolify não encontra a imagem

**Causa:** Variável DOCKER_USERNAME não configurada ou imagem não existe

**Solução:**
1. Verifique se a imagem existe no Docker Hub
2. Configure DOCKER_USERNAME no Coolify
3. Tente fazer pull manual: `docker pull seu-usuario/opensheets:latest`

---

## Limpeza de Imagens Antigas

No Docker Hub (free tier tem limite de 1 repositório privado):

1. Vá em **Repositories** → `opensheets` → **Tags**
2. Delete tags antigas que não usa mais
3. Mantenha: `latest`, versões estáveis importantes

Ou use Docker Hub CLI:
```bash
# Instalar hub-tool
brew install docker/hub-tool/hub-tool  # macOS

# Listar tags
hub-tool tag ls seu-usuario/opensheets

# Deletar tag
hub-tool tag rm seu-usuario/opensheets:old-tag
```

---

## Recursos

- [Docker Hub](https://hub.docker.com/)
- [GitHub Actions - Docker](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [Semantic Versioning](https://semver.org/)

---

## Próximos Passos

Depois de configurar:

1. ✅ Push código para GitHub → imagem publicada automaticamente
2. ✅ No Coolify, usar `docker-compose.prod.yml`
3. ✅ Deploy super rápido (apenas pull da imagem)
4. ✅ Criar releases com tags quando tiver versões estáveis
