# MODELO MENTAL DO DOCKER (ENTENDA ISSO OU NADA FAZ SENTIDO)

Docker tem **4 conceitos centrais**:

1️⃣ **Image** → blueprint (imutável)
2️⃣ **Container** → instância rodando da image
3️⃣ **Volume** → dados persistentes
4️⃣ **Network** → comunicação entre containers

Todo comando Docker mexe em **um desses quatro**. Se não souber qual, você está no escuro.

---

# COMANDOS QUE VOCÊ JÁ USOU (EXPLICADOS)

## `docker build`

```bash
docker build -t backend ./backend
```

### O que faz

* Lê o `Dockerfile`
* Executa cada instrução
* Gera uma **image**

### Flags importantes

* `-t backend` → nomeia a image
* `.` ou `./backend` → contexto de build

📌 **Regra**:

> Quanto maior o contexto, mais lento o build.

---

## `docker run`

```bash
docker run -p 8000:8000 backend
```

### O que faz

* Cria um **container**
* A partir da image `backend`
* Executa o `CMD`

### Flags essenciais

* `-p 8000:8000`

  * esquerda → host
  * direita → container

Sem isso, você não acessa nada de fora.

---

## `docker ps`

```bash
docker ps
```

### O que faz

* Lista containers **rodando**

```bash
docker ps -a
```

* Lista **todos** (parados também)

📌 Se não aparece aqui, não está rodando.

---

## `docker stop`

```bash
docker stop <container_id>
```

* Para o container de forma graciosa
* Envia `SIGTERM`

---

## `docker rm`

```bash
docker rm <container_id>
```

* Remove container parado
* Não remove image

📌 Container ≠ image. Nunca confunda.

---

# COMANDOS DE INSPEÇÃO (ESSENCIAIS PRA DEBUG)

## `docker logs`

```bash
docker logs <container_id>
```

* Exibe stdout/stderr do container
* Equivalente a “print” do backend

📌 80% dos bugs você resolve aqui.

---

## `docker exec`

```bash
docker exec -it <container_id> bash
```

### O que faz

* Entra dentro do container
* Sessão interativa

Usos reais:

* Rodar `python manage.py migrate`
* Inspecionar filesystem
* Testar import

📌 Não é produção, é debug.

---

## `docker inspect`

```bash
docker inspect <container_id>
```

* Mostra tudo:

  * env vars
  * mounts
  * network
  * IP interno

Quando nada faz sentido, isso responde.

---

# COMANDOS DE IMAGE (VOCÊ VAI USAR MUITO)

## `docker images`

```bash
docker images
```

Lista todas as images locais.

---

## `docker rmi`

```bash
docker rmi backend
```

Remove image.

📌 Só remove se **nenhum container** estiver usando.

---

## `docker system prune`

```bash
docker system prune
```

⚠️ **PERIGOSO SE NÃO SOUBER O QUE FAZ**

Remove:

* containers parados
* images órfãs
* networks não usadas
* cache de build

Use quando:

* Docker está ocupando 20GB
* Você sabe que pode reconstruir tudo

---

# VOLUMES (ONDE INICIANTES QUEBRAM TUDO)

## `docker volume ls`

```bash
docker volume ls
```

---

## `docker volume rm`

```bash
docker volume rm <volume_name>
```

📌 Banco de dados geralmente vive aqui.
Remover volume = **apagar dados**.

---

# NETWORK (VOCÊ USA SEM SABER)

## `docker network ls`

```bash
docker network ls
```

---

## `docker network inspect`

```bash
docker network inspect <network>
```

Mostra:

* containers conectados
* DNS interno

📌 Em docker-compose, containers se falam por **nome do serviço**.

---

# DOCKER COMPOSE (VOCÊ VAI USAR TODO DIA)

> `docker-compose` antigo
> `docker compose` moderno

Sempre use:

```bash
docker compose
```

---

## `docker compose up`

```bash
docker compose up
```

* Cria:

  * network
  * volumes
  * containers

```bash
docker compose up --build
```

Força rebuild.

---

## `docker compose down`

```bash
docker compose down
```

Para tudo.

```bash
docker compose down -v
```

⚠️ Remove volumes (banco vai junto)

---

## `docker compose logs`

```bash
docker compose logs -f
```

Logs de todos os serviços.

---

## `docker compose exec`

```bash
docker compose exec backend bash
```

Entrar no container via compose (melhor que `docker exec`).

---

# COMANDOS QUE SEPARAM AMADOR DE PROFISSIONAL

## `--rm`

```bash
docker run --rm backend
```

Remove container automaticamente ao parar.

---

## `--env / -e`

```bash
docker run -e DEBUG=1 backend
```

Variável de ambiente.

---

## `--name`

```bash
docker run --name api backend
```

Sem nome, Docker gera lixo tipo `angry_turing`.

---

# RESUMO BRUTAL (O QUE VOCÊ PRECISA DOMINAR)

Se você souber **isso aqui**, você está acima da média:

### Core

* `docker build`
* `docker run`
* `docker ps`
* `docker logs`
* `docker exec`

### Limpeza

* `docker rm`
* `docker rmi`
* `docker system prune`

### Compose

* `docker compose up`
* `docker compose down`
* `docker compose logs`
* `docker compose exec`

Se você **não sabe explicar a diferença entre image e container**, volte.

---
