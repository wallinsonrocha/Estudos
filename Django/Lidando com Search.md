# Django
As pesquisas são partes primordiais de para buscar dados.

## Url
Usaremos isso para colocar no form do nosso código que contém a a form de pesquisa. Para buscar o derecionamento de uma url. Pode ser usada dentro do form.
```
<form action="{% url 'recipes:search' %}" method="GET" class="search-form">
    <input type="search" class="search-input" name="q" required>
    <button type="submit" class="search-button"><i class="fas fa-search"></i></button>
</form>
```

## Mais uma função na `view.py`
Criaremos mais uma função na nossa aplicação para executar essa função.

```
def search(request):
    search_term = request.GET.get('q', '').strip()

    if not search_term:
        raise Http404()

    return render(request, 'recipes/pages/search.html')
```

> 'q' é a variável que irá retornar na url do form. repare que, em for, o input search nomeamos = 'q'.

Essa função irá retornar a requisição Get.

## Tornando a pesquisa mais funcional
O certo seria pesquisar por termos, no mínimo. Ou seja, alguma palavra chave que contém em algum dado que procuramos na base de dados.

Para isso podemos importar o "Q".
```
from django.db.models import Q
...

def search(request):
    search_term = request.GET.get('q', '').strip()

    if not search_term:
        raise Http404()

    recipes = Recipe.objects.filter(
        Q(title__icontains=search_term) |
        Q(description__icontains=search_term),
        Q(
            Q(title__icontains=search_term) |
            Q(description__icontains=search_term),
        ),
        is_published=True
    ).order_by('-id')

    return render(request, 'recipes/pages/search.html')
```

Esse 'Q' irá modificar o retorno das nossas pesquisas. Nesse caso ele servirá como um OR e um AND. Para definirmos um or colocamos `|`. Caso um and, não precisamos especificar nada. Isso serme para deixar o código mais enxuto.

[Fazendo consultas complexas com Q](https://docs.djangoproject.com/pt-br/3.2/topics/db/queries/#complex-lookups-with-q-objects)

[Filtros de campo](https://docs.djangoproject.com/pt-br/3.2/topics/db/queries/#field-lookups)