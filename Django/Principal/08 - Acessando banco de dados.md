# Django

## Acessando

Em views, aprendemos que o que está lá será apresentado em nossa página. Então, para poder acessar o banco de dados e colocar as informações lá, faremos:

**views.py**
```
...
from .models import Recipe

def home(request):
    recipes = Recipe.objects.all().order_by('-id')
    return render(request, 'caminho', context={
        'recipes': recipes,
    })
```

Caso desejarmos acessar os atributos de uma **foreign key** nós faremos:
```
def home(request, category_id):
    recipes = Recipe.objects.filter(
        category__id = category_id # Irá receber o argumento category_id.
        ).order_by('-id')
    return render(request, 'caminho', context={
        'recipes': recipes,
    })
```