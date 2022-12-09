# Deploy

## Statics
Após os passos anterioores devemos sincronizar o nosso projeto local com o servidor. Logo em seguida ativamos o ambiente virtual e fazemos um collectstatic.

```
cd app_repo/
.venv/bin/activate
python manage.py collectstatic
```