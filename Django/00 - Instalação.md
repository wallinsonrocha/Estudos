# Django

## Instalação
```
pip install django
```

Caso seja necessário atualizar o pip
```
pip install pip --upgrade
```

## Iniciar projeto django
Precisamos, primeiramente, gerar os arquivos que serão utilizados para utilizar o Django.
```
django-admin startproject projeto .
```
Esse ponto serve para indicar que o projeto deve ser criado na mesma pasta que estamos utilizando. Assim será criado um arquivo chamado "manage.py"


Para iniciar o servidor em modo de desenvolvimento:
```
python manage.py runserver
```