# Django

## Passos
### **Criar nosso app**
```
python manage.py startapp nome
```

### **Incluir a aplicação no INSTALlED_APPS no `settings.py` do projeto.**
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Podemos por aqui
    'nome_do_app',
]
```

### **Criar um arquivo `urls.py` dentro da pasta da aplicação criada para importar as views.**
```
# Exemplo:
from django.urls import path
from . import views


urlpatterns = [
    path('register/', views.register_view, name='register'), #caminho, view e nome
]
```
### **Não esquecer de criar um views para por na url da nossa aplicação em `view.py`**
```
# Exemplo:
from django.shortcuts import render


def register_view(request):
    return render(request, 'authors/pages/register_view.html')
```

### **Criar as seguintes pastas e arquivos**

Criar pastas:

    - templates
        - nome_do_app
            - pages
                - nome_da_pagina.html
            - partials
                - nome_do_partial.html

> partial é a parte que colocamos as partes dos códigos que serão colocadas nas páginas.

[Caso nescessário trabalhar com arquivos estáticos](https://github.com/wallinsonrocha/Estudos/blob/master/Django/Principal/02%20-%20Renderizando%20HTML%2C%20vari%C3%A1veis%20de%20contexto%20e%20gerenciamento%20de%20arquivos%20est%C3%A1ticos.md#arquivos-est%C3%A1ticos)

### Colocá-lo no `urls.py` do projeto
```
from django.urls import include, path
...
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('recipes.urls')),
]
```