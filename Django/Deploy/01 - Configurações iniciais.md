# Deploy

## Configuração no .env e DATABASES no `settings.py`

Em nossa variável de ambiente podemos colocar algumas variáveis. Aquelas que estão comentadas estão desativadas. Para usar basta algumas modificações. Não obstante, isso irá depender do banco que usaremos.

```
# Database settings
# Sqlite
DATABASE_ENGINE = 'django.db.backends.sqlite3'
DATABASE_NAME = "./db.sqlite3"

# Postgres
# DATABASE_ENGINE='django.db.backends.postgresql'
# DATABASE_NAME="basededados"

DATABASE_USER = "usuario"
DATABASE_PASSWORD = "senha"
DATABASE_HOST = "127.0.0.1"
DATABASE_PORT = "5432"
```

Já em nosso `settings.py` configuraremos a área de DATABASE para se comunicar com as variáveis de ambiente. Não podemos esquecer, é claro, de [configurar as variáveis de ambiente.](https://github.com/wallinsonrocha/Estudos/blob/master/Django/Vari%C3%A1veis%20de%20ambiente.md).
```
DATABASES = {
    'default': {
        'ENGINE': os.environ.get('DATABASE_ENGINE'),
        'NAME': os.environ.get('DATABASE_NAME'),
        'USER': os.environ.get('DATABASE_USER'),
        'PASSWORD': os.environ.get('DATABASE_PASSWORD'),
        'HOST': os.environ.get('DATABASE_HOST'),
        'PORT': os.environ.get('DATABASE_PORT'),
    }
}
```

## Outras modificações

Devemos **desabilitar o debug** do nosso .env para que em nosso `settings` o degud seja desabilitado.
```
DEBUG = 0
```

Direcionar o domínio ou ip da nossa aplicação.
> Isso será feito no `settings.py`.
```
ALLOWED_HOST: list[str] = ['nomedodominio.com']
```