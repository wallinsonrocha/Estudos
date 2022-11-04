# Django

## Admin
O django tem uma area de administração. Lá podemos manipular a aplicação. Para acessar, precisamos, primeiramente, criar um super usuário para ter o login e a senha.
```
python manage.py createsuperuser
```

Ao criar, podemos ter acesso a essa área.

## Configurações no arquivo admin
Assim como o model, precisamos registrar as classes para o admin. Dessa forma ele será reconhecido.
```
from django.contrib import admin
from . models import Category

class CategoryAdmin(admin.ModelAdmin):
    ...

admin.site.register(Category, CategoryAdmin)
```

Há uma outra forma de fazer esse registro.
```
@admin.register(Recipe)
class RecipeAdmin(admin.ModelAdmin)
```