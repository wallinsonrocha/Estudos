# Django

Através do django roles nós poderemos editar as permissões de cada usuário do nosso sistema.

## Instalação
```
pip install django-role-permissions
```

## Configurações

- ### Incluir o `rolepermissions` nos INSTALLED_APP como se fosse um app.
- ### Criar uma variável em `settings.py` e definir o local onde o nosso arquivo será guardado.
```
ROLEPERMISSIONS_MODULE = "pasta.roles" // O nome role pode ser outro. Ele será o nosso arquivo.
```

## Dentro do arquivo criado.

Prosseguindo, iremos configurar o nosso arquivo.
```
from rolepermissions.roles import AbstractUserRole

class Gerente(AbstractUserRole):
    // A variável abaixo personaliza algumas permissões
    available_permissions = {'criar_metas' :True, 'ver_metas': True}

class Vendedor(AbstractUserRole):
    available_permissions = {'acessar_produtos' :True, 'ver_metas': True}
```

## Atribuindo grupos

Dentro da view nós podemos extender `assign_role` de `rolepermissions` para definir o grupo do nosso usuário.
```
from django.contrib.auth.models import User
from rolepermissions.role import assign_role

def criar_usuario(request):
    user = User.objects.create_user(username="caio", password="1234")
    user.save()
    assign_role(user, 'gerente')
    return HttpResponse('Usuário salvo com sucesso')
```

## Restringindo acesso para determinados grupos

```
from rolepermissions.decorators import has_role_decorator

@has_role_decorator('gerente')
def home(request):
    return HttpResponse('Estou na home')    
```

## Revogando ou atribuindo permissões
```
from rolepermissions.permissions import revoke_permission, grant_permission

def home(request):

    revoke_permission(request.user, 'ver_metas') // request.user é o usuário logado.
    grant_permission(request.user, 'ver_metas')

    return HttpResponse('Estou na home')    
```

## Ver o grupo do usuário

```
from rolepermissions.role import get_user_roles

def home(request):
    print(get_user_roles(request.user))
    return HttpResponse('Estou na home')
```

## Colocando tags no html
```
{% load permissions_tags %}

{% if user|has_role:'gerente' %}
    Página do gerente
{% endif %}
```

Outra forma mais direta com permissões é:

```
{% load permissions_tags %}

{% if user|can:'ver_metas' %}
    Página de metas
{% endif %}
```