# Django admin - personalização do template

Esse tutorial é dedicado a todos aqueles que decidiram usar o django admin para o gerênciamento do seu site. Nele você aprenderá a personalizar o template "home" do django e, também, a criar várias páginas de administração para diferentes coisas.

## Mudando o template do django.contrib.admin (padrão)

Para fazer isso, no meu projeto eu criei dois arquivos que não são criadas. Fiz isso para questões de organização. `admin.py` e `apps.py`.

### `admin.py`
```
from django.contrib.admin import AdminSite


class CustomAdminSite(AdminSite):
    site_header = "Meu Site Admin Customizado"
    site_title = "Meu Site Admin Customizado"
    index_title = "Página inicial do Meu Site Admin Customizado"
    index_template = 'admin/custom_home.html'

```

**Observação extremamente importante:** Você deve certificar que o seu TEMPLATES em `settings.py` em DIRS esteja configurada. A minha estava assim:
```
'DIRS': [
            BASE_DIR / 'base_templates',
        ],
```

No meu caso, `index_template = 'admin/custom_home.html'` será procurado na pasta "base_templates".


### `apps.py`
```
from django.contrib.admin.apps import AdminConfig


class CustomAdminConfig(AdminConfig): 
    default_site = 'project.admin.CustomAdminSite' 
```


### Configurando settings.py
Em `INSTALLED_APPS` você irá substituir `django.contrib.admin,` por `'project_name.apps.CustomAdminConfig',`.

### Editando o template
No meu arquivo `custom_home.html` coloquei a seguinte configuração:
```
{% extends "admin/base_site.html" %}

{% block content %}
<div class="custom-index">
  <h1>Bem-vindo ao meu site personalizado de administração Django!</h1>
  <p>Esta é a minha página inicial personalizada.</p>
</div>
{% endblock %}
```

Feito isso o seu admin estará diferente.

---

## Criando mais uma área de admin

Para a criação personalizada de mais uma eu criei o objeto `custom_admin_site`, como vocês podem observar abaixo.
```
from django.contrib.admin import AdminSite


class CustomAdminSite(AdminSite):
    site_header = "Meu Site Admin Customizado"
    site_title = "Meu Site Admin Customizado"
    index_title = "Página inicial do Meu Site Admin Customizado"
    index_template = 'admin/custom_home.html'

custom_admin_site = CustomAdminSite(name='meu_admin')

```

Ele será utilizado para ser colocado no arquivo `urls.py` de sua preferência.

`urls.py`
```
from project.admin import custom_admin_site # local onde o objeto está

urlpatterns = [
    ...
    path('meu_admin/', custom_admin_site.urls),
    ...
]
```

Por ser uma outra área, o formato padrão de colocá-lo no admin fará com que ele apareça apenas no admin, mas não na outra área de administração (no caso "meu_admin/").

Para mais um model adicionar podemos utilizar o método register(). Observe o exemplo abaixo na última linha:

`admin.py`
```
class CustomAdminSite(AdminSite):
    site_header = "Meu Site Admin Customizado"
    site_title = "Meu Site Admin Customizado"
    index_title = "Página inicial do Meu Site Admin Customizado"
    index_template = 'admin/custom_home.html'

custom_admin_site = CustomAdminSite(name='meu_admin')
custom_admin_site.register(Food)
```

Nesse caso estou adicionando o model Food.

---

## Limitar alguns Adiministradores a determinadas páginas

Nós podemos fazer esse trabalho criando algum model que possua um campo role/função (depende como você chamar).
```
from django.contrib.auth import get_user_model
from django.db import models

User = get_user_model()

class UserInfos(models.Model):    
    OPTION_CHOICES = [
        ('cliente', 'Cliente'),
        ('gerente', 'Gerente'),
        ('delivery', 'Entregador')
    ]
    user = models.OneToOneField(User, on_delete=models.CASCADE, default=None)
    number = models.CharField(max_length=20, blank=True, null=True)
    first_login = models.BooleanField(default=True)
    role = models.CharField(max_length=20, choices=OPTION_CHOICES, default='cliente')

    def __str__(self):
        return f'{self.user}'    
```

Com essas configurações nós podemos criar um `middleware`. Você pode criar esse arquivo no seu projeto e colocar o seguinte:
```
from django.shortcuts import redirect
from django.urls import reverse

from user_infos.models import UserInfos


def has_role(user, role):
    if user is None or not user.is_authenticated:
        return False
    try:
        user_info = UserInfos.objects.get(user=user)
        if user_info.role == role:
            return True
        else:
            return False
    except UserInfos.DoesNotExist:
        return False


class CustomGerenteMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        if request.path.startswith('/gerente/'):
            if not has_role(request.user, 'gerente'):
                return redirect(reverse('food:home'))
        response = self.get_response(request)
        return response
```

Dessa forma somente o usuário que tiver a função de gerente poderá acessar essa página. Mas, para isso funcionar, nós devemos colocá-lo em no MIDDLEWARE em `setting.py`:
```
MIDDLEWARE = [
    ...
    'project.middleware.CustomGerenteMiddleware' # Caminho para CustomGerenteMiddleware
]
```

## Permitiondo que não Adms possam entrar dependendo da sua função

Para isso você deve fazer a alteração manual como administrador. Há diversos vídeos na internete ensinando. Mas, caso você prefira, pode criar alguma function para fazer isso automaticamente.

---

## Criar mais actions e modificar o template de algum model

Você pode personalizar o template de um modelo no Django admin, definindo uma nova classe que estende ModelAdmin e substituindo o atributo change_form_template.

Por exemplo, para personalizar o template para o modelo MyModel, você pode definir uma nova classe MyModelAdmin que estende ModelAdmin e substitui o atributo change_form_template com o nome do seu novo template.

Aqui está um exemplo de como personalizar o template de um modelo no Django admin:

```
from django.contrib import admin
from .models import MyModel

class MyModelAdmin(admin.ModelAdmin):
    change_list_template = 'myapp/custom_change_form.html'

admin.site.register(MyModel, MyModelAdmin)
```

E, no html, você pode organizar assim:
```
{% extends 'admin/change_list.html' %}

{% block object-tools %}
<div class="custom-index">
  <h1>Bem-vindo ao meu site personalizado de administração Django!</h1>
  <p>Esta é a minha página inicial personalizada.</p>
</div>
{{ block.super }}
{% endblock %}
```

---

## Criar mais actions
Você pode adicionar mais actions personalizadas no Django Admin usando a propriedade actions na sua classe ModelAdmin. A propriedade actions é uma lista de funções que executam uma ação personalizada quando selecionadas a partir da lista de objetos na interface do Django Admin.

```
def say_hello(modeladmin, request, queryset):
    queryset.update(title='Olá')
    

# Register your models here.
@admin.register(models.Food)
class FoodAdmin(admin.ModelAdmin):
    actions = [say_hello]
```

Nesse caso, todos os itens selecionados terão o seu titulo mudado para "Olá".

---

## Recomendação de estudo

Você pode estudar um pouco mais através desse [site](https://books.agiliq.com/projects/django-admin-cookbook/en/latest/custom_button.html).