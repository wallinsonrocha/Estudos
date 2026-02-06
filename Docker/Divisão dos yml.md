Como rodar (sem confusão)

``DEV``
```
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

``PROD``
```
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

`docker-compose.yml`
```
services:
  db:
    image: postgres:16
    container_name: coihub-db
    env_file:
      - backend/.env
    volumes:
      - postgres_data:/var/lib/postgresql/data

  backend:
    build: ./backend
    env_file:
      - backend/.env
    ports:
      - "8000:8000"
    depends_on:
      - db

volumes:
  postgres_data:
```

`docker-compose.dev.yml`
```
services:
  backend:
    volumes:
      - ./backend:/app
    command: python manage.py runserver 0.0.0.0:8000
    ports:
      - "8000:8000"
```

`docker-compose.prod.yml`
```
services:
  backend:
    container_name: coihub-backend
    command: gunicorn app.wsgi:application --bind 0.0.0.0:8000
    ports:
      - "8000:8000"
```