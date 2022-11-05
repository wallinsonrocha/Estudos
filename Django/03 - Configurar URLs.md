# Django

## IDs na URL
Para configurar um ID na nossa url, devemos fazer as configurações no nosso arquivo urls.py da aplicação.
```
from . import views # views, aqui, é o arquivo.

urlpatterns = [
    path('', views.home),
    path('recipes/<int:id>', views.recipes, name="recipe"),
]
```

Ao especificar o id como int, dizemos que na url só será aceita numeros inteiros para aquela parte da url. Além disso, na nossa view em recipes, devemos implementar mais um argumento: id.
```
def recipe(request, id):
    return render(request, 'caminho_do_arquivo_html', context={
        'name': 'Teste'
    })
```

Para entender mais um pouco, [clique aqui.](https://docs.djangoproject.com/pt-br/3.2/topics/http/urls/)