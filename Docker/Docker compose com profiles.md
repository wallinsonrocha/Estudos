```
services:
  db:
    image: postgres:16
    container_name: coihub-db
    env_file:
      - backend/.env
    volumes:
      - postgres_data:/var/lib/postgresql/data
    profiles: ["dev", "prod"]

  backend-dev:
    build: ./backend
    env_file:
      - backend/.env
    volumes:
      - ./backend:/app
    command: python manage.py runserver 0.0.0.0:8000
    ports:
      - "8000:8000"
    depends_on:
      - db
    profiles: ["dev"]

  backend-prod:
    build: ./backend
    container_name: coihub-backend
    env_file:
      - backend/.env
    command: gunicorn app.wsgi:application --bind 0.0.0.0:8000
    ports:
      - "8000:8000"
    depends_on:
      - db
    profiles: ["prod"]

volumes:
  postgres_data:
```

---

```
docker compose --profile dev up
```

```
docker compose --profile prod up -d
```