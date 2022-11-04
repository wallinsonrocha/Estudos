# Django

## Render
Dentro do nosso arquivo view em um App temos a exportação de um render que server para fazer a renderização HTML através do Django. O render exige, no mínimo, dois argumentos: request e o caminho do template. 
```
from django.http import HttpResponse
from django.shortcuts import render

def home(request):
    return render(request, 'recipes/home.html')
```

Este caminho devemos criar. Para isso, recomenda-se a criação de uma pasta **templates** no nosso app e outra pasta com o nome da aplicação. Após isso, basta criarmos um arquivo html e editá-lo. Isso é feito para evitar conflito com os arquivos da pasta base_templates.

#### {%include% 'caminho'}
Uma das outras funcionalidades do django é a capacidad de dividir o HTML. Supomos que fora criado um arquivo HTML que queremos dividir em partes por estar muito grande. Criaremos uma pasta chamada Pages, que quardará a pagina principal e, após isso, também criaremos a pasta Partial que guardará partes do nosso código.

Para incluirmos o parcial na pagina principal, usamos:
```
{% include 'caminho' %}
```

## Arquivos estáticos
Para implementarmos arquivos estáticos no nosso HTML, iremos, primeiramente, habilitá-lo e depois direcionar o caminho dele. 

Importação no arquivo HTML.
```
{% load static %}
```

Implementação:
```
<link rel="stylesheet"> href="{% static 'recipes/css/style.css' %}"
```