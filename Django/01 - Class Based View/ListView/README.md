# Django

## Aqui estão alguns dos principais atributos de uma ListView no Django:

template_name: O nome do template que a ListView irá utilizar para renderizar os dados. Por padrão, o nome do template deve seguir a convenção "model_list.html", onde "model" é o nome do modelo em minúsculo. Por exemplo, se o modelo for Product, o nome do template padrão seria "product_list.html".
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
```

model: O modelo que será utilizado para recuperar os objetos a serem exibidos na lista.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
```
queryset: Uma consulta personalizada que é usada para obter os objetos a serem exibidos na lista. Se o queryset for fornecido, o model será ignorado.
Exemplo:
```
class ProductListView(ListView):
    template_name = "products.html"

    def get_queryset(self):
        return Product.objects.filter(is_available=True)
```
context_object_name: O nome da variável de contexto que contém a lista de objetos a serem exibidos no template. Por padrão, essa variável é chamada de object_list.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
    context_object_name = "products"
```
paginate_by: O número de objetos por página. Quando a lista é longa, é comum usar a paginação para dividir a lista em várias páginas. O valor padrão é None, o que significa que a paginação não será usada.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
    paginate_by = 10
```
ordering: A ordem em que os objetos são exibidos na lista. O valor padrão é None, o que significa que a ordem será determinada pelo modelo.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
    ordering = "-created_at"
```
paginate_orphans: O número mínimo de objetos necessários na última página. Isso é útil quando a paginação é usada para evitar páginas com apenas alguns objetos.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
    paginate_by = 10
    paginate_orphans = 3
```
allow_empty: Se definido como False, a ListView retornará um erro 404 quando a lista estiver vazia.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"
    allow_empty = False
```
context_data: Um dicionário de dados adicionais para serem adicionados ao contexto do template.
Exemplo:
```
class ProductListView(ListView):
    model = Product
    template_name = "products.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["title"] = "Lista de Produtos"
        return context
```

## Funções de ListView

As `ListViews` do Django são uma forma conveniente de exibir um conjunto de dados em uma lista. Algumas das funções que elas fornecem são:

- **`model`**: define o modelo a ser usado para recuperar os objetos a serem exibidos.

- **`template_name`**: define o nome do template a ser usado para renderizar a lista.

- **`queryset`**: define o conjunto de objetos a serem exibidos na lista.

- **`context_object_name`**: define o nome da variável de contexto que contém a lista de objetos.

- **`paginate_by`**: define o número de objetos a serem exibidos por página, habilitando a paginação.

- **`ordering`**: define a ordem em que os objetos são exibidos na lista.

- **`get_context_data()`**: adiciona dados adicionais ao contexto do template.

- **`get_queryset()`**: personaliza o queryset para recuperar os objetos desejados.

- **`get()`**: manipula solicitações GET.

- **`post()`**: manipula solicitações POST.

- **`put()`**: manipula solicitações PUT.

- **`delete()`**: manipula solicitações DELETE.


## Base para home, search e category.
```
from django.views.generic import ListView


PER_PAGE = int(os.environ.get('PER_PAGE', 6))


class RecipeListViewBase(ListView):
    # Atributos mais importantes que devemos editar
    model = Recipe
    context_object_name = 'recipes'
    ordering = ['-id']
    template_name = 'recipes/pages/home.html'

    # Para acessar a queryset
    # Dentro da variável qs há uma função do mesmo nome. Por isso devemos colocar o super().
    # Através dela podemos fazer a manipulação do que será exebido, por exemplo.
    def get_queryset(self, *args, **kwargs):
        qs = super().get_queryset(*args, **kwargs)
        qs = qs.filter(
            is_published=True,
        )
        return qs


    # Aqui foi utilizado para a paginação.
    # Utilizado para manipular o contexto.
    def get_context_data(self, *args, **kwargs):
        # O mesmo definido em context_object_name
        ctx = super().get_context_data(*args, **kwargs)
        page_obj, pagination_range = make_pagination(
            self.request,
            ctx.get('recipes'),
            PER_PAGE
        )
        ctx.update(
            {'recipes': page_obj, 'pagination_range': pagination_range}
        )
        return ctx
```

## Category
```
class RecipeListViewCategory(RecipeListViewBase):
    template_name = 'recipes/pages/category.html'

    def get_queryset(self, *args, **kwargs):
        qs = super().get_queryset(*args, **kwargs)
        qs = qs.filter(
            # category_id é referente ao que será passado com request
            category__id=self.kwargs.get('category_id')
        )
        return qs
```

## Search
```
class RecipeListViewSearch(RecipeListViewBase):
    template_name = 'recipes/pages/search.html'

    def get_queryset(self, *args, **kwargs):
        search_term = self.request.GET.get('q', '')
        qs = super().get_queryset(*args, **kwargs)
        qs = qs.filter(
            Q(
                Q(title__icontains=search_term) |
                Q(description__icontains=search_term),
            )
        )
        return qs

    def get_context_data(self, *args, **kwargs):
        ctx = super().get_context_data(*args, **kwargs)
        search_term = self.request.GET.get('q', '')

        ctx.update({
            'page_title': f'Search for "{search_term}" |',
            'search_term': search_term,
            'additional_url_query': f'&q={search_term}',
        })

        return ctx
```
