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
