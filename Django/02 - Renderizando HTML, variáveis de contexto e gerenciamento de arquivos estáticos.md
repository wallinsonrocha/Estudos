# Django

## Render
Dentro do nosso arquivo **view** em um App temos a exportação de um render que server para fazer a renderização HTML através do Django. O render exige, no mínimo, dois argumentos: request e o caminho do template. 
```
from django.http import HttpResponse
from django.shortcuts import render

def home(request):
    return render(request, 'recipes/home.html')
```

Este caminho devemos criar. Para isso, recomenda-se a criação de uma pasta **templates** no nosso app e outra pasta com o nome da aplicação. Após isso, basta criarmos um arquivo html e editá-lo. Isso é feito para evitar conflito com os arquivos da pasta base_templates.

> Obs.: Não podemos esquecer de colocar a aplicação no nosso arquivo **settings**.

## Partials
Através dele nós podemos dividir os códigos da nossa aplicação.

### {%include% 'caminho'}
Supomos que fora criado um arquivo HTML que queremos dividir em partes por estar muito grande. Criaremos, dentro de **templates/nomeApp** uma pasta chamada **pages**, que quardará a pagina principal e, após isso, também criaremos a pasta **partial** que guardará partes do nosso código.

Dentro da pasta **partial** iremos criar os nossos arquivos parciais. Nele estará parte do código que será incluída no final.

Exemplo com o header.
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
```

Para incluirmos o parcial na pagina principal, usamos o {% include 'caminho' %}. Exemplo:
```
{% include 'recipes/partials/head.html' %}
```

## {{name_variavel}}
Através de {{variavel}} podemos colocar um valor que está armazenado em nosso contexto na view.

**view.py**
```
def home(request):
    return render(request, 'caminho', context={
        'name': 'Wallinson',
    })
```

**Exemplo com o header**
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{name}}</title>
</head>
```

Dessa forma no título estará 'Wallinson'

## Arquivos estáticos
Para implementarmos arquivos estáticos no nosso HTML, iremos habilitá-lo e depois direcionar o caminho dele. 

Mas, antes disso, é importante criarmos uma pasta chamada **static/nomeApp** (cria-se primeiro a pasta static e, dentro dela, nomeApp) para armazenar o nosso CSS e nossas imagens. E, dentro dela, iremos criar uma pasta chamada **media** para imagens e outra chamada **css**.
> Obs.: Ela deve ser criada fora de **templates**.

Importação no arquivo HTML. Geralmente na primeira linha do código.
```
{% load static %}
```

Implementação:
```
<link rel="stylesheet"> href="{% static 'recipes/css/style.css' %}"
```

## STATIC_URL e STATICFILES_DIRS
Em um dado momento, poderemos trabalhar com arquivos estáticos que não estarão atrelados a nenhum app. Quando isso ocorre nós os armazenamos em uma pasta fora da aplicação. Geralmente criamos a pasta com o prefixo **base_**. Após isso, dentro dela, criamos uma pasta **global** para armazenar os arquivos de acordo com a sua preferência.

Supomos que foi criado um arquivo chamado **global-styles.css** dentro da pasta **base_static/global/css**.

Para que esse caminho possa ser reconhecido, devemos adicionar mais algumas configurações em **settings.py**.
>Preferencialmente abaixo de STATIC_URL
```
STATICFILES_DIRS = [
    BASE_DIR / 'base_static',
]
```

A **importação no arquivo html** ficaria assim:
```
<link rel="stylesheet"> href="{% static 'global/css/glocal-styles.css' %}"
```

Essa divisão é muito importante para evitar a colisão de nomes.

## STATIC_ROOT
Podemos aproveitar para criar uma configuração avançada quando enviarmos para o serividor. No mesmo arquivo **settings.py** podemos adicionar:
```
STATIC_ROOT = BASE_DIR / 'static'
```

Quando os arquivos estaticos forem coletados, eles irão para essa pasta que será criada.
Podemos coletá-los através do seguinte código:
```
python manage.py collectstatic
```