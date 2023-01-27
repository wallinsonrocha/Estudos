# Rest Framework

## CORS
Lidar com outros dominios possam que acessar a nossa API.

## Configuração
**Pip**
```
python -m pip install django-cors-headers
```

## Settings

**Instaleds App**
```
INSTALLED_APPS = [
    ...
    "corsheaders",
    ...
]
```

**Middleware**
```
MIDDLEWARE = [
    ...
    'corsheaders.middleware.CorsMiddleware',
    ...
]
```

**Criar**
```
CORS_ALLOWED_ORIGINS = [
    'http://127.0.0.1:5500', // Esse domínio é um exemplo.
]
```

## Configuração no .env

**Cors allowed origins**
```
from utils.environment import get_env_variable, parse_comma_sep_str_to_list

CORS_ALLOWED_ORIGINS = parse_comma_sep_str_to_list(
    get_env_variable('CORS_ALLOWED_ORIGINS')
)
```

**.env**
```
ALLOWED_HOSTS = '127.0.0.1, localhost,'
CORS_ALLOWED_ORIGINS = 'http://127.0.0.1:5500,'
```