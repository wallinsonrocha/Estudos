# 1️⃣ ANTES DE ESCREVER QUALQUER LINHA: MODELO MENTAL

Um Dockerfile responde **4 perguntas**, nessa ordem:

1. **Em que ambiente isso vai rodar?**
2. **O que precisa ser instalado?**
3. **Onde o código vai viver?**
4. **Como a aplicação inicia?**

Se você não sabe responder essas perguntas, você **não está pronto** para escrever um Dockerfile.

---

# 2️⃣ ESTRUTURA MÍNIMA DE UM DOCKERFILE

Todo Dockerfile sério tem, no mínimo:

```
FROM
ENV
WORKDIR
COPY
RUN
CMD ou ENTRYPOINT
```

Não é dogma, é prática.

---

# 3️⃣ PASSO A PASSO: CONSTRUINDO UM DOCKERFILE REAL (DJANGO)

Vou montar **linha por linha**, explicando o *porquê*, não só o *o quê*.

---

## 🔹 PASSO 1 — BASE IMAGE (`FROM`)

```dockerfile
FROM python:3.12-slim
```

### O que você está decidindo aqui

* Linguagem → Python
* Versão → 3.12
* Sistema → Debian slim

### Por que isso importa

* Tudo que vem depois depende disso
* Trocar base = trocar tudo

📌 **Regra prática**:
Python → `python:x.y-slim`
Não use Alpine se não souber exatamente por quê.

---

## 🔹 PASSO 2 — VARIÁVEIS DE AMBIENTE (`ENV`)

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
```

### O que isso resolve

* Evita arquivos `.pyc`
* Logs aparecem em tempo real

📌 Docker = logs em stdout.
Sem isso, debug vira inferno.

---

## 🔹 PASSO 3 — DIRETÓRIO DE TRABALHO (`WORKDIR`)

```dockerfile
WORKDIR /app
```

### O que isso faz

* Cria (se não existir) `/app`
* Define como diretório padrão

📌 Nunca trabalhe em `/`.
Isso não é VM dos anos 2000.

---

## 🔹 PASSO 4 — DEPENDÊNCIAS PRIMEIRO (CACHE INTELIGENTE)

```dockerfile
COPY pyproject.toml uv.lock ./
```

### Por que copiar só isso primeiro?

Docker usa **cache por camada**.

* Código muda sempre
* Dependência muda pouco

Essa ordem deixa o build:

* mais rápido
* previsível
* profissional

📌 Esse é o erro nº1 de iniciantes.

---

## 🔹 PASSO 5 — INSTALAR DEPENDÊNCIAS (`RUN`)

```dockerfile
RUN pip install --no-cache-dir uv
RUN uv sync --frozen
```

### O que acontece

* Instala o gerenciador de dependência
* Instala exatamente o que está no lockfile

📌 `--frozen` = build determinístico
Se quebrar aqui, o problema é seu projeto, não o Docker.

---

## 🔹 PASSO 6 — COPIAR O CÓDIGO (`COPY`)

```dockerfile
COPY . .
```

Agora sim:

* backend
* settings
* apps
* scripts

Tudo entra no container.

---

## 🔹 PASSO 7 — COMANDO DE INICIALIZAÇÃO (`CMD` ou `ENTRYPOINT`)

### Opção simples (CMD)

```dockerfile
CMD ["gunicorn", "app.wsgi:application", "--bind", "0.0.0.0:8000"]
```

* Define o comando padrão
* Pode ser sobrescrito no compose

### Quando usar ENTRYPOINT?

Quando você **precisa de lógica antes de iniciar**
(ex: esperar banco, rodar migração)

Exemplo:

```dockerfile
ENTRYPOINT ["./entrypoint.sh"]
```

📌 CMD = comando
📌 ENTRYPOINT = comportamento

---

# 4️⃣ DOCKERFILE FINAL (O QUE VOCÊ DEVERIA TER)

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY pyproject.toml uv.lock ./

RUN pip install --no-cache-dir uv
RUN uv sync --frozen

COPY . .

CMD ["gunicorn", "app.wsgi:application", "--bind", "0.0.0.0:8000"]
```

Isso é:

* limpo
* previsível
* profissional

---

# 5️⃣ ERROS CLÁSSICOS QUE VOCÊ DEVE EVITAR

❌ Instalar dependência depois de copiar código
❌ Usar `ADD` sem motivo
❌ Usar `latest` como tag
❌ Misturar banco no mesmo container
❌ Colocar segredo direto no Dockerfile

Dockerfile **não é lugar de segredo**.

---
