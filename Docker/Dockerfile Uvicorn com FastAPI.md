```
FROM python:3.12-slim

WORKDIR /app

RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

RUN curl -Ls https://astral.sh/uv/install.sh | sh

ENV PATH="/root/.local/bin:$PATH"

RUN uv --version

COPY pyproject.toml uv.lock ./
RUN uv sync --frozen

COPY . .

CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Isso é importante `ENV PATH="/root/.local/bin:$PATH"`

---

# Explicação

## Linha 1

```dockerfile
FROM python:3.12-slim
```

Mesma ideia de antes:

* Python oficial
* Versão fixa
* Imagem enxuta

Nada de especial aqui.

---

## Linha 3

```dockerfile
WORKDIR /app
```

Cria e entra no diretório `/app`.

Tudo padrão.
Até aqui, Dockerfile genérico.

---

## Linha 5

```dockerfile
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```

Aqui começa algo importante.

### O que está acontecendo:

* Atualiza lista de pacotes
* Instala `curl`
* Remove cache do apt

### Por quê?

Porque você **não está usando pip para instalar o uv**.
Você vai baixar o **install script oficial do uv** via HTTP.

Sem `curl`, isso quebra.

---

## Linha 7

```dockerfile
RUN curl -Ls https://astral.sh/uv/install.sh | sh
```

Aqui você instala o `uv` **do jeito recomendado pelo próprio projeto**.

O script:

* Baixa o binário
* Instala em:

  ```
  /root/.local/bin/uv
  ```

⚠️ Isso é crítico para entender o próximo passo.

---

## Linha 9 — O ponto-chave

```dockerfile
ENV PATH="/root/.local/bin:$PATH"
```

### Isso NÃO é opcional.

Sem isso, o Dockerfile **não funciona**.

---

### O problema real

O script do `uv` instala o binário em:

```
/root/.local/bin
```

Esse diretório **NÃO está no PATH padrão do Docker**.

PATH padrão costuma ser algo como:

```
/usr/local/bin:/usr/bin:/bin
```

Ou seja:

```bash
uv --version
# command not found
```

---

### O que essa linha faz

```dockerfile
ENV PATH="/root/.local/bin:$PATH"
```

Ela diz:

> “Antes de procurar comandos no sistema, procure em `/root/.local/bin`.”

Agora:

* `uv` é encontrado
* `uv sync` funciona
* `uv run` funciona

Sem isso:

* Todas as linhas seguintes que usam `uv` quebram

---

### Analogia direta

Essa linha é o equivalente Docker de:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Mas:

* Persistente
* Funciona em qualquer shell
* Correto para container

---

## Linha 11

```dockerfile
RUN uv --version
```

Linha de **verificação explícita**.

Serve para:

* Garantir que o `uv` foi instalado corretamente
* Falhar cedo no build se algo deu errado

Boa prática.

---

## Linha 13

```dockerfile
COPY pyproject.toml uv.lock ./
```

Mesmo padrão correto de cache:

* Copia só arquivos de dependência
* Evita reinstalar dependências quando só o código muda

---

## Linha 14

```dockerfile
RUN uv sync --frozen
```

Aqui o `uv`:

* Cria o virtualenv (por padrão em `.venv`)
* Instala dependências exatamente como no lockfile
* `--frozen` garante reprodutibilidade

Até aqui, tudo certo.

⚠️ **Mas observe algo importante**:
Você **não ajustou o PATH para o `.venv`**.

Isso é uma decisão arquitetural — já falo disso.

---

## Linha 16

```dockerfile
COPY . .
```

Copia o código depois das dependências.
Correto para cache.

---

## Linha 18

```dockerfile
CMD ["uv", "run", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Aqui está a maior diferença conceitual desse Dockerfile.

---

### O que `uv run` faz

`uv run`:

* Ativa o virtualenv **temporariamente**
* Executa o comando **dentro do ambiente correto**
* Não depende de `PATH` do `.venv`

Ou seja:

```bash
uv run uvicorn ...
```

é equivalente a:

```bash
source .venv/bin/activate
uvicorn ...
```

Só que:

* Sem shell
* Sem gambiarra
* Determinístico

---

## Comparação direta com o outro Dockerfile

### Dockerfile anterior

* Você expunha o `.venv/bin` no `PATH`
* Gunicorn rodava diretamente

### Esse Dockerfile

* Você **não mexe no PATH do virtualenv**
* Sempre executa via `uv run`

São dois modelos válidos.

---

## Agora, a crítica honesta (coach mode)

### 👍 Pontos fortes

* Instala `uv` do jeito oficial
* Não depende de pip
* `uv run` evita erros de PATH
* Muito explícito e previsível

---

### ⚠️ Pontos questionáveis

1. **Rodar sempre via `uv run`**

   * Em produção, isso adiciona uma camada a mais
   * É levemente mais lento
   * Não é necessário depois do build

2. **Virtualenv não integrado ao sistema**

   * Debug via `docker exec` fica menos intuitivo
   * `python`, `uvicorn`, etc não estão direto no PATH

3. **Instalação via script remoto**

   * Depende de rede
   * Menos previsível que instalar binário fixo
   * Em ambientes ultra-controlados, isso é um problema

---

## Quando esse Dockerfile faz MAIS sentido

* Projetos FastAPI
* Times que querem padronizar tudo via `uv`
* Ambientes onde ninguém quer pensar em PATH
* Dev containers

---

## Quando eu NÃO usaria isso

* Produção crítica
* Containers que precisam ser minimalistas
* Ambientes onde startup time importa muito
* Quando quero controle total do runtime

---

## Verdade final

Essa linha:

```dockerfile
ENV PATH="/root/.local/bin:$PATH"
```

É:

* O que torna o `uv` utilizável
* O elo entre o sistema e o runtime
* Absolutamente obrigatória

Sem ela, **todo o Dockerfile é inválido**.
