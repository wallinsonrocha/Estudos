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

## Criando apps
Os apps são respostas do servidor quando acessada determinada URI. No arquivo urls.py há uma função chamada admin que executa uma função de administração. Poderíamos colocar mais funções lá que funcionariam como app, não obstante ficaria muito cheio e desorganizado. Para resolver isso, portanto, criamos os App.

```
python manage.py startapp nome
```

Será criado uma pasta com vários arquivos. As funções que seriam criadas no arquivo urls, serão criadas em view. Dentro dela iremos fazer alguns imports para que a função possa ser executada e criá-las.
```
from django.http import HttpResponse

def sobre(request):
    return HttpResponse('Sobre')
```

### Urls
As únicas coisas que poderão ir para esse aquivo "urls" serão as rotas dentro de urlpatterns. Logo, as funções criadas no arquivo view serão referenciadas na url para que a requisição seja respondida. Mas, para melhorar nosso código, podemos criar outro arquivo urls no app que criamos e fazer uma inclusão no arquico urls principal.

#### No arquivo urls.py da aplicação criada
```
from django.urls import path
from nomeApp.views import sobre, contato

...
urlpatterns = [
    path('admin', admin.site.urls),
    path('sobre/', sobre),
    path('sobre/', contato)
]
```

#### No arquivo urls.py principal
```
from django.urls import include, path
...
urlpatterns = [
    path('admin', admin.site.urls),
    path('', include('nomeDoApp.urls'))
]
```

Neste útlimo código, o **""** em **path('', ...)** indica qual será a uri que ele deve procurar começando a partir de uma determinada rota. Como não há nada isso significa que não há parâmetros. Mas, caso houvesse, poderia ser, por exemplo: dominio/nomeApp/sobre

#### Configurando o arquivo settings.py
Neste arquivo há uma seção de Apps instalados. Nele podemos encontrar o admin, auth, contenttypes... É nesse mesmo lugar que iremos colocar o nome do nosso app. Supomos que criamos um app chamado 'recipes'. 
```
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'recipes',  
]
```