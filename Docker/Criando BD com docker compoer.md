Algumas coisas podem ser feitas já no `.env`. Podemos aproveitar para colocar as variáveis de ambiente junto com o backend.

O depends_on, na linha final faz o container depender de `db`, nesse caso.

`docke-compose.yml`
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
    container_name: coihub-backend
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

`.env`
```
# Django
DEBUG=1
SECRET_KEY=super-secret-key-dev
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1

# Database
POSTGRES_DB=backend_db
POSTGRES_USER=django
POSTGRES_PASSWORD=django123
POSTGRES_HOST=db
POSTGRES_PORT=5432
```