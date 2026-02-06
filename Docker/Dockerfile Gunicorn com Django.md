```
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY pyproject.toml uv.lock ./

RUN pip install --no-cache-dir uv
RUN uv sync --frozen

ENV PATH="/app/.venv/bin:$PATH"

COPY . .

CMD ["gunicorn", "app.wsgi:application", "--bind", "0.0.0.0:8000"]
```

Aqui está o segredo: `ENV PATH="/app/.venv/bin:$PATH"`

---

# Explicação

## Linha 1

```dockerfile
FROM python:3.12-slim
```

* Usa a imagem oficial do Python
* `3.12` → versão explícita (reprodutibilidade)
* `slim` → remove coisas inúteis (docs, debug tools, etc.)

**Tradeoff real**:

* 👍 imagem menor
* 👎 você precisa instalar dependências de sistema manualmente se precisar (ex: `libpq-dev`)

---

## Linhas 3–4

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
```

### `PYTHONDONTWRITEBYTECODE=1`

* Impede criação de `.pyc`
* Evita sujeira no container
* Não traz ganho de performance relevante, é organização

### `PYTHONUNBUFFERED=1`

* Logs saem em tempo real
* Sem isso, Gunicorn/Django pode “segurar” logs
* **Essencial para Docker**, senão você fica cego

---

## Linha 6

```dockerfile
WORKDIR /app
```

* Cria (se não existir) e entra no diretório `/app`
* Tudo daqui pra baixo é relativo a `/app`

Sem isso:

* Você teria que usar caminhos absolutos o tempo todo
* Dockerfile ficaria feio e frágil

---

## Linha 8

```dockerfile
COPY pyproject.toml uv.lock ./
```

Aqui tem **engenharia de build**, não acaso.

* Copia **somente** arquivos de dependência
* NÃO copia o código ainda

### Por quê?

Porque Docker faz cache por camada.

Se o código muda mas:

* `pyproject.toml`
* `uv.lock`

não mudam → **dependências não são reinstaladas**

Isso acelera builds drasticamente.

---

## Linha 10

```dockerfile
RUN pip install --no-cache-dir uv
```

* Instala o `uv`
* `--no-cache-dir` evita lixo na imagem

Sim, você usa `pip` só para instalar o `uv`.
Depois disso, **pip nunca mais entra em cena**.

---

## Linha 11

```dockerfile
RUN uv sync --frozen
```

Essa linha é **crítica**.

O que acontece aqui:

1. `uv` lê:

   * `pyproject.toml`
   * `uv.lock`
2. Cria um **virtualenv isolado**
3. Instala **exatamente** as versões do lockfile

### `--frozen`

* Falha se o `pyproject.toml` não bater com o `uv.lock`
* Garante builds determinísticos
* Isso é padrão de produção

Sem isso:

* Seu container pode instalar versões diferentes em cada build

---

## Linha 13 — O “SEGREDO”

```dockerfile
ENV PATH="/app/.venv/bin:$PATH"
```

Agora atenção. Aqui está o ponto que separa quem **entende** de quem só copia tutorial.

---

### O que o `uv` faz por padrão?

O `uv` cria o virtualenv em:

```
/app/.venv
```

E os binários ficam em:

```
/app/.venv/bin/
```

Exemplos:

```
/app/.venv/bin/python
/app/.venv/bin/gunicorn
/app/.venv/bin/django-admin
```

---

### O problema REAL

Quando o container roda:

```bash
gunicorn app.wsgi:application
```

O sistema operacional procura o binário seguindo o `PATH`.

Sem essa linha, o PATH padrão é algo como:

```
/usr/local/bin:/usr/bin:/bin
```

👉 **O `gunicorn` instalado no virtualenv NÃO está lá.**

Resultado:

* `gunicorn: command not found`
* Ou pior: usa um gunicorn global (se existir)

---

### O que essa linha faz exatamente

```dockerfile
ENV PATH="/app/.venv/bin:$PATH"
```

Ela diz:

> “Antes de procurar comandos no sistema, procure dentro do virtualenv.”

Agora:

* `python` → vem do `.venv`
* `gunicorn` → vem do `.venv`
* `pip` → vem do `.venv`

Você **não precisa** fazer:

```bash
source .venv/bin/activate
```

Em Docker isso é inútil.
Docker não tem shell interativo persistente.

---

### Verdade dura

Essa linha é o equivalente Docker de:

```bash
source .venv/bin/activate
```

Só que:

* Funciona
* É determinística
* Não depende de shell

Sem ela, seu Dockerfile está **conceitualmente errado**, mesmo que “funcione por acaso”.

---

## Linha 15

```dockerfile
COPY . .
```

Agora sim você copia o código.

Por que **depois** do `uv sync`?

* Para não invalidar o cache de dependências
* Código muda mais que dependência

Boa prática absoluta.

---

## Linha 17

```dockerfile
CMD ["gunicorn", "app.wsgi:application", "--bind", "0.0.0.0:8000"]
```

* Usa `exec form` (array JSON)
* Não passa por shell
* Gunicorn vira PID 1 (bom para sinais)

### `0.0.0.0`

* Obrigatório em Docker
* `127.0.0.1` **não expõe porta**

---

## Resumo brutalmente honesto

Esse Dockerfile:

✔ Usa imagem enxuta
✔ Tem cache eficiente
✔ Usa `uv` corretamente
✔ Usa virtualenv do jeito certo
✔ Não depende de `activate`
✔ É production-ready

### O ponto mais importante

Sem esta linha:

```dockerfile
ENV PATH="/app/.venv/bin:$PATH"
```

Todo o setup com `uv` **desmorona conceitualmente**.

Ela é:

* O elo entre o virtualenv e o container
* O motivo pelo qual seus comandos funcionam
* A diferença entre entender e copiar
