# Django

## Preparação
Recomenda-se criar um novo app chamado authors para personalizar as permissões e os tipos de usuários separadamente.
```
python manage.py startapp authors
```

Após isso basta seguirmos as configurações padrões para configurar o nosso app ([ver aqui]()), seguiremos as seguintes recomendações:

### Em `urls.py` do projeto iremos reconfigurá-lo dessa forma:
```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('recipes.urls')),
    path('authors/', include('authors.urls')),
]
```

Para author iremos colocar um caminho padrão antes de fazer qualquer direcionamento. Ou seja será `site.com/authors/alguma_coisa/`.

### View

Modelo de view

```
def register_view(request):
    return render(request, 'authors/pages/register_view.html')
```