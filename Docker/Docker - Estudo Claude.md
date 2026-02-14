#  Guia Completo: Docker para Desenvolvimento Backend

##  Índice

1. [Introdução ao Docker](#introdução-ao-docker)
2. [Conceitos Fundamentais](#conceitos-fundamentais)
3. [Dockerfile - Criando Imagens](#dockerfile---criando-imagens)
4. [Docker Compose - Orquestrando Containers](#docker-compose---orquestrando-containers)
5. [Multi-Stage Builds](#multi-stage-builds)
6. [Profiles - Ambientes Diferentes](#profiles---ambientes-diferentes)
7. [Volumes e Persistência](#volumes-e-persistência)
8. [Networks](#networks)
9. [Variáveis de Ambiente](#variáveis-de-ambiente)
10. [Boas Práticas](#boas-práticas)
11. [Comandos Essenciais](#comandos-essenciais)
12. [Debugging e Troubleshooting](#debugging-e-troubleshooting)

---

## Introdução ao Docker

### O que é Docker?

Docker é uma plataforma para **criar, distribuir e executar aplicações em containers**.

**Container** = Pacote que contém:
- Sua aplicação
- Todas as dependências
- Sistema de arquivos mínimo
- Configurações

**Analogia:** Container é como uma "caixa" auto-suficiente que roda em qualquer lugar.

### Por que usar Docker?

```
 SEM DOCKER:
Desenvolvedor: "Funciona na minha máquina! "
Produção: Quebra tudo 

 COM DOCKER:
Desenvolvedor: Cria container
Produção: Usa o MESMO container
Resultado: Funciona igual em todos os lugares! 
```

**Benefícios:**

1. **Portabilidade** - Roda em qualquer lugar (Windows, Mac, Linux, Cloud)
2. **Isolamento** - Cada container é independente
3. **Reprodutibilidade** - Ambiente idêntico em dev/test/prod
4. **Facilidade** - Um comando sobe toda a stack
5. **Eficiência** - Mais leve que VMs

---

## Conceitos Fundamentais

### Imagem vs Container

```
┌─────────────────────────────────────────────┐
│              IMAGEM                         │
│  - Template read-only                       │
│  - Criada a partir do Dockerfile            │
│  - Como uma "receita de bolo"               │
│  - Exemplo: python:3.11                     │
└─────────────────────────────────────────────┘
                    ↓
            docker run (cria)
                    ↓
┌─────────────────────────────────────────────┐
│             CONTAINER                       │
│  - Instância executável da imagem           │
│  - Como um "bolo pronto"                    │
│  - Pode ter múltiplos containers da mesma   │
│    imagem                                   │
└─────────────────────────────────────────────┘
```

**Exemplo prático:**
```bash
# Imagem = Classe
# Container = Objeto/Instância

# Baixa imagem do Python
docker pull python:3.11

# Cria 3 containers da mesma imagem
docker run python:3.11  # Container 1
docker run python:3.11  # Container 2
docker run python:3.11  # Container 3
```

### Docker Registry

```
Docker Hub (registry público)
    ↓
Você faz: docker pull python:3.11
    ↓
Imagem baixada para sua máquina
    ↓
docker run cria container
```

---

## Dockerfile - Criando Imagens

### Anatomia de um Dockerfile

```dockerfile
# Dockerfile

# 1. IMAGEM BASE
FROM python:3.11-slim

# 2. METADADOS
LABEL maintainer="seu-email@example.com"
LABEL version="1.0"

# 3. VARIÁVEIS DE BUILD
ARG APP_DIR=/app

# 4. VARIÁVEIS DE AMBIENTE
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

# 5. DIRETÓRIO DE TRABALHO
WORKDIR ${APP_DIR}

# 6. COPIAR ARQUIVOS
COPY requirements.txt .

# 7. EXECUTAR COMANDOS
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# 8. COPIAR CÓDIGO
COPY . .

# 9. EXPOR PORTA
EXPOSE 8000

# 10. COMANDO PADRÃO
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Explicação Detalhada

#### 1. FROM - Imagem Base

```dockerfile
# Imagem oficial do Python (versão slim = menor)
FROM python:3.11-slim

# Outras opções:
# FROM python:3.11-alpine  # Ainda menor (usa Alpine Linux)
# FROM python:3.11         # Versão completa (maior)
```

**Escolha baseada em:**
- `slim` → Bom equilíbrio (recomendado para maioria)
- `alpine` → Menor tamanho (pode ter problemas com algumas libs)
- `full` → Tudo incluído (maior, use só se necessário)

#### 2. LABEL - Metadados

```dockerfile
LABEL maintainer="dev@example.com"
LABEL version="1.0.0"
LABEL description="API FastAPI em produção"
```

**Uso:** Documentação, filtros, automação

#### 3. ARG - Variáveis de Build

```dockerfile
# ARG só existe durante o BUILD
ARG PYTHON_VERSION=3.11
ARG APP_DIR=/app

FROM python:${PYTHON_VERSION}-slim
WORKDIR ${APP_DIR}
```

**Diferença ARG vs ENV:**
```
ARG  → Disponível apenas no build
ENV  → Disponível no build E no container em execução
```

#### 4. ENV - Variáveis de Ambiente

```dockerfile
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    DATABASE_URL=postgresql://localhost/db

# PYTHONUNBUFFERED=1       → Logs em tempo real
# PYTHONDONTWRITEBYTECODE=1 → Não gera .pyc
# PIP_NO_CACHE_DIR=1       → Não cacheia downloads (menor imagem)
```

#### 5. WORKDIR - Diretório de Trabalho

```dockerfile
WORKDIR /app

# Equivalente a:
# RUN mkdir -p /app
# CD /app

# Todos os comandos seguintes rodam em /app
```

#### 6. COPY vs ADD

```dockerfile
# COPY - Recomendado (simples)
COPY requirements.txt .
COPY app/ ./app/

# ADD - Faz mais (extrai tar, baixa URLs)
ADD https://example.com/file.tar.gz .  # Baixa e extrai
```

**Regra:** Use `COPY` sempre. Use `ADD` só se precisar das features extras.

#### 7. RUN - Executa Comandos

```dockerfile
#  ERRADO - Cria muitas camadas
RUN apt-get update
RUN apt-get install -y curl
RUN pip install --upgrade pip
RUN pip install -r requirements.txt

#  CERTO - Uma camada, limpa cache
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/* && \
    pip install --upgrade pip && \
    pip install -r requirements.txt
```

**Por quê?**
- Cada `RUN` cria uma camada (layer)
- Camadas aumentam tamanho da imagem
- Limpar cache reduz tamanho

#### 8. EXPOSE - Documenta Porta

```dockerfile
EXPOSE 8000

# IMPORTANTE: EXPOSE é apenas documentação!
# Não publica a porta automaticamente
# Use -p no docker run para publicar
```

#### 9. CMD vs ENTRYPOINT

```dockerfile
# CMD - Comando padrão (pode ser sobrescrito)
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0"]

# ENTRYPOINT - Sempre executa (não sobrescreve)
ENTRYPOINT ["python"]
CMD ["app.py"]  # Argumento padrão para ENTRYPOINT

# Juntos:
# docker run → python app.py
# docker run outro.py → python outro.py
```

**Quando usar:**
- `CMD` → Comando completo que pode ser substituído
- `ENTRYPOINT` → Executável fixo + `CMD` como args

---

### Exemplo Completo: Dockerfile para FastAPI

```dockerfile
# ==========================================
# Dockerfile para FastAPI + PostgreSQL
# ==========================================

# Imagem base
FROM python:3.11-slim as base

# Metadados
LABEL maintainer="dev@myapp.com"
LABEL version="1.0.0"

# Variáveis de ambiente
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# Diretório de trabalho
WORKDIR /app

# Instalar dependências do sistema
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        postgresql-client \
        gcc \
        && rm -rf /var/lib/apt/lists/*

# Copiar requirements e instalar dependências Python
COPY requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

# Copiar código da aplicação
COPY . .

# Criar usuário não-root (segurança)
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

# Mudar para usuário não-root
USER appuser

# Expor porta
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')"

# Comando padrão
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

##  Docker Compose - Orquestrando Containers

### O que é Docker Compose?

Ferramenta para **definir e rodar aplicações multi-container**.

**Exemplo:** Sua aplicação precisa de:
- App FastAPI
- PostgreSQL
- Redis
- Nginx

Com Docker Compose, **um comando sobe tudo**:
```bash
docker compose up
```

### Anatomia do docker-compose.yml

```yaml
# docker-compose.yml

# Versão do formato (3.8 é atual e estável)
version: '3.8'

# Definição dos serviços (containers)
services:
  
  # Serviço 1: Banco de dados
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  # Serviço 2: Aplicação FastAPI
  app:
    build: .
    command: uvicorn app.main:app --host 0.0.0.0 --reload
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://myuser:mypassword@db:5432/mydb

# Volumes nomeados (persistência)
volumes:
  postgres_data:
```

### Explicação Detalhada

#### 1. services - Definição de Containers

```yaml
services:
  app:           # Nome do serviço (você escolhe)
    image: ...   # OU
    build: ...   # Uma das duas opções
```

**Diferença:**
```yaml
# OPÇÃO 1: Usar imagem pronta
app:
  image: python:3.11

# OPÇÃO 2: Buildar a partir do Dockerfile
app:
  build:
    context: .              # Onde está o Dockerfile
    dockerfile: Dockerfile  # Nome do arquivo (padrão: Dockerfile)
    args:                   # ARGs do Dockerfile
      PYTHON_VERSION: 3.11
```

#### 2. ports - Mapeamento de Portas

```yaml
ports:
  - "8000:8000"  # HOST:CONTAINER
  - "5432:5432"

# 8000:8000 significa:
# - Porta 8000 no host → Porta 8000 no container
# - Acessa via localhost:8000
```

**Formatos:**
```yaml
ports:
  - "8000:8000"      # Bind em todas as interfaces
  - "127.0.0.1:8000:8000"  # Bind apenas em localhost
  - "8000"           # Porta aleatória no host → 8000 no container
```

#### 3. volumes - Persistência e Bind Mounts

```yaml
volumes:
  # Bind mount (compartilha pasta do host com container)
  - .:/app                              # Pasta atual → /app no container
  - ./data:/app/data                    # Subpasta data
  
  # Volume nomeado (gerenciado pelo Docker)
  - postgres_data:/var/lib/postgresql/data
  
  # Volume anônimo
  - /app/node_modules                   # Não sobrescreve
```

**Diferenças:**
```
BIND MOUNT:
  - Aponta para pasta real no host
  - Mudanças refletem imediatamente
  - Uso: desenvolvimento (hot reload)

VOLUME NOMEADO:
  - Gerenciado pelo Docker
  - Persiste entre recriações
  - Uso: banco de dados, uploads

VOLUME ANÔNIMO:
  - Temporário
  - Uso: evitar sobrescrever node_modules, etc
```

#### 4. environment - Variáveis de Ambiente

```yaml
environment:
  # Formato 1: Chave-valor
  DATABASE_URL: postgresql://user:pass@db/mydb
  DEBUG: "true"
  
  # Formato 2: Lista
  - DATABASE_URL=postgresql://user:pass@db/mydb
  - DEBUG=true

# OU carrega de arquivo .env
env_file:
  - .env
  - .env.local
```

#### 5. depends_on - Ordem de Inicialização

```yaml
app:
  depends_on:
    - db
    - redis

# IMPORTANTE: depends_on NÃO espera o serviço estar PRONTO
# Apenas garante ordem de START

# Para esperar serviço estar pronto:
depends_on:
  db:
    condition: service_healthy  # Requer healthcheck
```

#### 6. networks - Redes Personalizadas

```yaml
services:
  app:
    networks:
      - backend
  
  db:
    networks:
      - backend
  
  nginx:
    networks:
      - frontend
      - backend

networks:
  frontend:
  backend:
```

**Por quê?**
- Isolamento (frontend não acessa db diretamente)
- Segurança
- Organização

---

### Exemplo Completo: Stack FastAPI

```yaml
# docker-compose.yml

version: '3.8'

services:
  # ==========================================
  # PostgreSQL Database
  # ==========================================
  db:
    image: postgres:15-alpine
    container_name: myapp_db
    restart: unless-stopped
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_DB: ${DB_NAME:-myapp}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Script inicial
    ports:
      - "${DB_PORT:-5432}:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - backend

  # ==========================================
  # Redis Cache
  # ==========================================
  redis:
    image: redis:7-alpine
    container_name: myapp_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    networks:
      - backend

  # ==========================================
  # FastAPI Application
  # ==========================================
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        PYTHON_VERSION: 3.11
    container_name: myapp_api
    restart: unless-stopped
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
    volumes:
      - .:/app                    # Código (hot reload)
      - /app/__pycache__          # Não compartilha cache
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://${DB_USER:-postgres}:${DB_PASSWORD:-postgres}@db:5432/${DB_NAME:-myapp}
      REDIS_URL: redis://redis:6379/0
      DEBUG: ${DEBUG:-true}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
      - frontend

  # ==========================================
  # Nginx Reverse Proxy
  # ==========================================
  nginx:
    image: nginx:alpine
    container_name: myapp_nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - frontend

# ==========================================
# Volumes
# ==========================================
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

# ==========================================
# Networks
# ==========================================
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

---

## ️ Multi-Stage Builds

Multi-stage builds criam imagens **menores e mais seguras**.

### Conceito

```
Stage 1 (builder): Instala tudo, compila, testa
    ↓
Stage 2 (final): Copia APENAS o necessário
    ↓
Resultado: Imagem final SEM dependências de build
```

### Exemplo Básico

```dockerfile
# ==========================================
# STAGE 1: Builder
# ==========================================
FROM python:3.11-slim as builder

WORKDIR /app

# Instalar dependências de build
RUN apt-get update && \
    apt-get install -y gcc python3-dev

# Copiar requirements
COPY requirements.txt .

# Instalar em diretório temporário
RUN pip install --user --no-cache-dir -r requirements.txt

# ==========================================
# STAGE 2: Final (imagem de produção)
# ==========================================
FROM python:3.11-slim

WORKDIR /app

# Copiar apenas dependências instaladas (do stage builder)
COPY --from=builder /root/.local /root/.local

# Copiar código
COPY . .

# PATH para encontrar pacotes
ENV PATH=/root/.local/bin:$PATH

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0"]
```

**Resultado:**
```
Imagem builder: 1.5 GB (com gcc, dev tools)
Imagem final:   200 MB (só runtime)
```

### Exemplo Avançado: Build, Test, Production

```dockerfile
# ==========================================
# BASE: Configurações comuns
# ==========================================
FROM python:3.11-slim as base

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# ==========================================
# BUILDER: Instala dependências
# ==========================================
FROM base as builder

RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc python3-dev && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt

# ==========================================
# DEVELOPMENT: Ambiente de desenvolvimento
# ==========================================
FROM base as development

COPY --from=builder /wheels /wheels
COPY requirements.txt .

RUN pip install --no-cache /wheels/* && \
    pip install pytest pytest-cov black flake8

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--reload"]

# ==========================================
# TEST: Executa testes
# ==========================================
FROM development as test

RUN pytest --cov=app --cov-report=term-missing

# ==========================================
# PRODUCTION: Imagem final mínima
# ==========================================
FROM base as production

# Copiar apenas wheels compilados
COPY --from=builder /wheels /wheels
COPY requirements.txt .

RUN pip install --no-cache /wheels/*

# Criar usuário não-root
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

COPY --chown=appuser:appuser . .

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health')" || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Uso:**
```bash
# Build desenvolvimento
docker build --target development -t myapp:dev .

# Build teste (para e falha se testes falharem)
docker build --target test -t myapp:test .

# Build produção
docker build --target production -t myapp:prod .

# Ou padrão (pega último stage)
docker build -t myapp .
```

---

##  Profiles - Ambientes Diferentes

Profiles permitem **ativar/desativar serviços** baseado no ambiente.

### Exemplo Básico

```yaml
version: '3.8'

services:
  # Sempre ativo
  app:
    build: .
    ports:
      - "8000:8000"

  # Apenas em desenvolvimento
  db-dev:
    image: postgres:15
    profiles: ["dev"]
    environment:
      POSTGRES_PASSWORD: dev123

  # Apenas em teste
  db-test:
    image: postgres:15
    profiles: ["test"]
    environment:
      POSTGRES_PASSWORD: test123
    tmpfs:
      - /var/lib/postgresql/data  # Banco em memória

  # Apenas em produção
  db-prod:
    image: postgres:15
    profiles: ["prod"]
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # Do .env
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Uso:**
```bash
# Desenvolvimento
docker compose --profile dev up

# Teste
docker compose --profile test up

# Produção
docker compose --profile prod up

# Múltiplos profiles
docker compose --profile dev --profile debug up
```

### Exemplo Completo: Dev, Test, Prod

```yaml
version: '3.8'

services:
  # ==========================================
  # APP - Sempre ativo
  # ==========================================
  app:
    build:
      context: .
      target: ${BUILD_TARGET:-production}
    container_name: myapp
    restart: unless-stopped
    ports:
      - "${APP_PORT:-8000}:8000"
    env_file:
      - .env
    depends_on:
      - db
      - redis
    networks:
      - app_network

  # ==========================================
  # DATABASE - Dev
  # ==========================================
  db:
    image: postgres:15-alpine
    profiles: ["dev", "test"]
    container_name: myapp_db_dev
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev123
      POSTGRES_DB: myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data
    networks:
      - app_network

  # ==========================================
  # DATABASE - Production (externo)
  # ==========================================
  # Em produção, geralmente usa banco externo (RDS, etc)
  # Mas se quiser rodar localmente:
  db-prod:
    image: postgres:15-alpine
    profiles: ["prod"]
    container_name: myapp_db_prod
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_prod_data:/var/lib/postgresql/data
    networks:
      - app_network

  # ==========================================
  # REDIS - Todos os ambientes
  # ==========================================
  redis:
    image: redis:7-alpine
    container_name: myapp_redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    networks:
      - app_network

  # ==========================================
  # MAILHOG - Apenas Dev/Test (fake SMTP)
  # ==========================================
  mailhog:
    image: mailhog/mailhog
    profiles: ["dev", "test"]
    container_name: myapp_mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI
    networks:
      - app_network

  # ==========================================
  # PGADMIN - Apenas Dev
  # ==========================================
  pgadmin:
    image: dpage/pgadmin4
    profiles: ["dev"]
    container_name: myapp_pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    networks:
      - app_network

  # ==========================================
  # TESTS - Apenas Test Profile
  # ==========================================
  test:
    build:
      context: .
      target: test
    profiles: ["test"]
    container_name: myapp_test
    command: pytest -v --cov=app
    depends_on:
      - db
      - redis
    environment:
      DATABASE_URL: postgresql://dev:dev123@db:5432/myapp_test
      REDIS_URL: redis://redis:6379/1
    networks:
      - app_network

volumes:
  postgres_dev_data:
  postgres_prod_data:

networks:
  app_network:
    driver: bridge
```

**Comandos por ambiente:**

```bash
# DESENVOLVIMENTO
docker compose --profile dev up -d
# Sobe: app, db, redis, mailhog, pgadmin

# TESTE
docker compose --profile test up --abort-on-container-exit
# Sobe: app, db, redis, mailhog, test (para após testes)

# PRODUÇÃO
docker compose --profile prod up -d
# Sobe: app, db-prod, redis
```

---

##  Volumes e Persistência

### Tipos de Volumes

```yaml
services:
  app:
    volumes:
      # 1. BIND MOUNT - Pasta do host
      - ./app:/app                    # Desenvolvimento
      - ./logs:/app/logs              # Logs persistidos
      
      # 2. VOLUME NOMEADO - Gerenciado pelo Docker
      - postgres_data:/var/lib/postgresql/data
      
      # 3. VOLUME ANÔNIMO - Temporário
      - /app/node_modules             # Não sobrescreve
      
      # 4. TMPFS - Na memória (não persiste)
      - type: tmpfs
        target: /tmp

volumes:
  postgres_data:    # Define volume nomeado
```

### Quando Usar Cada Tipo

```yaml
# DESENVOLVIMENTO - Bind Mount
volumes:
  - .:/app           # Hot reload, edição em tempo real

# PRODUÇÃO - Volume Nomeado
volumes:
  - app_data:/app/data     # Persiste, backups fáceis

# CACHE - Volume Anônimo
volumes:
  - /app/node_modules      # Não compartilha, evita conflitos

# TEMPORÁRIO - Tmpfs
tmpfs:
  - /tmp                   # RAM, rápido, não persiste
```

### Backup e Restore

```bash
# BACKUP de volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz /data

# RESTORE de backup
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine sh -c "cd /data && tar xzf /backup/backup.tar.gz --strip 1"
```

---

##  Networks

### Tipos de Networks

```yaml
networks:
  # 1. BRIDGE (padrão) - Comunicação entre containers
  frontend:
    driver: bridge
  
  # 2. HOST - Usa rede do host (sem isolamento)
  host_network:
    driver: host
  
  # 3. NONE - Sem rede
  isolated:
    driver: none
```

### Exemplo de Isolamento

```yaml
version: '3.8'

services:
  # Frontend - Acessa apenas backend
  web:
    image: nginx
    networks:
      - frontend
  
  # Backend - Acessa frontend e database
  api:
    build: .
    networks:
      - frontend
      - backend
  
  # Database - Isolado, só backend acessa
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

**Resultado:**
```
web → pode comunicar com → api
api → pode comunicar com → web, db
db  → pode comunicar com → api
web → NÃO pode comunicar → db  Segurança!
```

### DNS Interno

```yaml
services:
  app:
    # ...
  db:
    # ...

# Dentro do container 'app':
# - db:5432 funciona!
# - O nome do serviço vira hostname
```

**Exemplo:**
```python
# app.py
DATABASE_URL = "postgresql://user:pass@db:5432/mydb"
#                                      ↑
#                              Nome do serviço!
```

---

##  Variáveis de Ambiente

### Precedência

```
1. docker-compose.yml (environment)
2. .env file
3. Variáveis de ambiente do shell
4. Dockerfile ENV
```

### .env File

```bash
# .env
DB_USER=postgres
DB_PASSWORD=secret123
DB_NAME=myapp
DB_PORT=5432
DEBUG=true
```

```yaml
# docker-compose.yml
services:
  app:
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:${DB_PORT}/${DB_NAME}
      DEBUG: ${DEBUG}
```

### env_file vs environment

```yaml
services:
  app:
    # OPÇÃO 1: Arquivo .env
    env_file:
      - .env
      - .env.local    # Sobrescreve .env
    
    # OPÇÃO 2: Direto no compose
    environment:
      DEBUG: true
      API_KEY: ${API_KEY}  # Do .env ou shell
```

### Múltiplos Ambientes

```
.env.dev
.env.test
.env.prod
```

```bash
# Desenvolvimento
docker compose --env-file .env.dev up

# Teste
docker compose --env-file .env.test up

# Produção
docker compose --env-file .env.prod up
```

---

##  Boas Práticas

### 1. Dockerfile

```dockerfile
#  CERTO

# Use imagens oficiais e específicas
FROM python:3.11-slim

# Combine comandos RUN
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# .dockerignore
# Crie arquivo .dockerignore:
__pycache__
*.pyc
.git
.env
.venv

# Copie requirements primeiro (cache de layers)
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .  # ← Só depois copia código

# Use multi-stage para imagens menores
FROM python:3.11-slim as builder
# ... build ...
FROM python:3.11-slim as final
COPY --from=builder /app /app

# Usuário não-root
RUN useradd -m appuser
USER appuser

# Health check
HEALTHCHECK CMD curl --fail http://localhost:8000/health || exit 1
```

```dockerfile
#  ERRADO

# Imagem genérica
FROM python

# Muitos RUN (muitas layers)
RUN apt-get update
RUN apt-get install curl
RUN pip install flask

# Sem .dockerignore (copia .git, venv, etc)

# Copia tudo antes de instalar deps
COPY . .
RUN pip install -r requirements.txt
# ↑ Invalida cache sempre que QUALQUER arquivo muda

# Root user (inseguro)
# (não especifica USER)

# Sem health check
```

### 2. docker-compose.yml

```yaml
#  CERTO

version: '3.8'

services:
  app:
    # Use variáveis de ambiente
    image: myapp:${VERSION:-latest}
    
    # Restart policy
    restart: unless-stopped
    
    # Health checks
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    
    # Recursos limitados
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          memory: 256M
    
    # Dependências com condições
    depends_on:
      db:
        condition: service_healthy
    
    # Networks explícitas
    networks:
      - backend

# Volumes nomeados (não bind mounts em prod)
volumes:
  postgres_data:

networks:
  backend:
```

```yaml
#  ERRADO

version: '3.8'

services:
  app:
    # Senha hardcoded
    environment:
      DB_PASSWORD: secret123
    
    # Sem restart policy
    # (container não reinicia se cair)
    
    # Sem health check
    
    # Sem limites de recursos
    # (pode consumir todo CPU/RAM)
    
    # depends_on sem condition
    depends_on:
      - db
    # ↑ App inicia antes do DB estar pronto
    
    # Bind mount em produção
    volumes:
      - .:/app
```

### 3. Segurança

```dockerfile
#  Segurança

# 1. Imagens oficiais
FROM python:3.11-slim

# 2. Scan de vulnerabilidades
# docker scan myapp:latest

# 3. Usuário não-root
RUN useradd -m -u 1000 appuser
USER appuser

# 4. Não expor informações sensíveis
# Use secrets/variáveis de ambiente
ENV API_KEY=${API_KEY}

# 5. Apenas o necessário
FROM python:3.11-slim as builder
# ... build ...
FROM python:3.11-slim  # ← Imagem limpa
COPY --from=builder /app /app  # ← Só artefatos
```

### 4. Tamanho da Imagem

```dockerfile
# Técnicas para reduzir tamanho:

# 1. Use imagens slim/alpine
FROM python:3.11-slim     # ~150MB
# vs
FROM python:3.11          # ~900MB

# 2. Multi-stage builds
FROM python:3.11 as builder
RUN pip install ...
FROM python:3.11-slim
COPY --from=builder ...

# 3. Limpe cache
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*  # ← Limpa

RUN pip install --no-cache-dir ...  # ← Sem cache

# 4. .dockerignore
__pycache__/
*.pyc
.git/
.venv/
tests/
```

**Resultado:**
```
Antes: 1.2 GB
Depois: 200 MB
```

---

## Comandos Essenciais

### Build

```bash
# Build simples
docker build -t myapp .

# Build com tag
docker build -t myapp:1.0.0 .

# Build sem cache
docker build --no-cache -t myapp .

# Build com target específico
docker build --target production -t myapp:prod .

# Build com argumentos
docker build --build-arg PYTHON_VERSION=3.11 -t myapp .
```

### Run

```bash
# Run básico
docker run myapp

# Run interativo
docker run -it myapp bash

# Run com porta
docker run -p 8000:8000 myapp

# Run em background
docker run -d myapp

# Run com nome
docker run --name myapp-container myapp

# Run com variáveis de ambiente
docker run -e DEBUG=true myapp

# Run com volume
docker run -v $(pwd):/app myapp

# Run com network
docker run --network mynetwork myapp

# Run com restart policy
docker run --restart unless-stopped myapp
```

### Compose

```bash
# Subir serviços
docker compose up

# Subir em background
docker compose up -d

# Subir com build
docker compose up --build

# Subir profile específico
docker compose --profile dev up

# Parar serviços
docker compose down

# Parar e remover volumes
docker compose down -v

# Ver logs
docker compose logs

# Logs de serviço específico
docker compose logs app

# Logs em tempo real
docker compose logs -f

# Executar comando
docker compose exec app bash

# Rodar comando one-off
docker compose run app python manage.py migrate
```

### Inspeção

```bash
# Listar containers
docker ps
docker ps -a  # Incluindo parados

# Listar imagens
docker images

# Listar volumes
docker volume ls

# Listar networks
docker network ls

# Inspecionar container
docker inspect <container>

# Ver logs
docker logs <container>
docker logs -f <container>  # Follow

# Ver estatísticas
docker stats
docker stats <container>

# Processos dentro do container
docker top <container>
```

### Limpeza

```bash
# Remover containers parados
docker container prune

# Remover imagens não usadas
docker image prune

# Remover volumes não usados
docker volume prune

# Remover networks não usadas
docker network prune

# Limpar TUDO (cuidado!)
docker system prune -a --volumes

# Ver espaço usado
docker system df
```

---

##  Debugging e Troubleshooting

### 1. Container não inicia

```bash
# Ver logs
docker compose logs app

# Ver últimas 100 linhas
docker compose logs --tail=100 app

# Logs em tempo real
docker compose logs -f app

# Inspecionar container
docker inspect myapp_container

# Ver eventos
docker events
```

### 2. Acessar container

```bash
# Shell interativo
docker compose exec app bash

# Se bash não existe (alpine)
docker compose exec app sh

# Como root (se precisa instalar algo)
docker compose exec -u root app bash

# Executar comando
docker compose exec app python --version
```

### 3. Rede não funciona

```bash
# Ver networks
docker network ls

# Inspecionar network
docker network inspect myapp_backend

# Testar conectividade dentro do container
docker compose exec app ping db
docker compose exec app nc -zv db 5432

# DNS
docker compose exec app nslookup db
```

### 4. Volume não persiste

```bash
# Listar volumes
docker volume ls

# Inspecionar volume
docker volume inspect postgres_data

# Ver onde está montado
docker inspect -f '{{.Mounts}}' container_name

# Acessar dados do volume
docker run --rm -v postgres_data:/data alpine ls -la /data
```

### 5. Build falha

```bash
# Build verboso
docker build --progress=plain --no-cache .

# Ver layers
docker history myapp

# Build step-by-step (para debug)
# Comente linhas do Dockerfile e vá descomentando
```

### 6. Container usa muita memória/CPU

```bash
# Ver estatísticas
docker stats

# Limitar recursos no compose
deploy:
  resources:
    limits:
      cpus: '0.5'
      memory: 512M

# Ver processos
docker top container_name
```

### 7. Conflito de portas

```bash
# Ver quem usa a porta
lsof -i :8000  # Linux/Mac
netstat -ano | findstr :8000  # Windows

# Mudar porta no compose
ports:
  - "8001:8000"  # Host 8001 → Container 8000
```

---

##  Checklist de Produção

```markdown
DOCKERFILE:
- [ ] Imagem base oficial e específica (python:3.11-slim)
- [ ] Multi-stage build
- [ ] .dockerignore configurado
- [ ] Usuário não-root
- [ ] Health check configurado
- [ ] Sem senhas/secrets hardcoded
- [ ] Camadas otimizadas (cache)
- [ ] Imagem < 500MB

DOCKER COMPOSE:
- [ ] Versão específica (3.8)
- [ ] Variáveis de ambiente no .env
- [ ] Restart policies configuradas
- [ ] Health checks em todos os serviços
- [ ] Depends_on com conditions
- [ ] Volumes nomeados (não bind mounts)
- [ ] Networks configuradas
- [ ] Limites de recursos definidos
- [ ] Profiles para ambientes diferentes

SEGURANÇA:
- [ ] Scan de vulnerabilidades (docker scan)
- [ ] Sem root user
- [ ] Secrets via variáveis de ambiente
- [ ] Networks isoladas
- [ ] Apenas portas necessárias expostas

PERFORMANCE:
- [ ] Imagem otimizada
- [ ] Cache de layers configurado
- [ ] Limites de recursos definidos
- [ ] Health checks não muito frequentes
```

---

##  Resumo Final

### Fluxo Completo

```
1. Escrever Dockerfile
   ↓
2. Criar .dockerignore
   ↓
3. Build da imagem
   ↓
4. Escrever docker-compose.yml
   ↓
5. Criar .env
   ↓
6. docker compose up
   ↓
7. Testar
   ↓
8. Deploy (docker compose --profile prod up)
```

### Comandos Mais Usados

```bash
# Desenvolvimento
docker compose --profile dev up          # Subir
docker compose logs -f app               # Ver logs
docker compose exec app bash             # Acessar
docker compose down                      # Parar

# Build
docker build -t myapp:latest .          # Buildar
docker compose up --build               # Buildar e subir

# Limpeza
docker compose down -v                  # Parar e limpar volumes
docker system prune -a                  # Limpar tudo

# Produção
docker compose --profile prod up -d     # Subir em background
docker compose logs --tail=100 -f       # Monitorar logs
```

### Arquivos Essenciais

```
projeto/
├── Dockerfile              # ← Como buildar imagem
├── .dockerignore          # ← O que NÃO copiar
├── docker-compose.yml     # ← Orquestração
├── .env                   # ← Variáveis de ambiente
└── .env.example          # ← Template do .env
```

---

**Versão:** 1.0 - Guia Completo de Docker
**Foco:** Backend Python/FastAPI
**Nível:** Iniciante → Avançado