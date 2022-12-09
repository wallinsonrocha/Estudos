# Django

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