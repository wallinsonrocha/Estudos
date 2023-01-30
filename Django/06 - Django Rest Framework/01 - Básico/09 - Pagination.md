# Rest Framework

## Configurações iniciais

`settings/environment.py` ou `settings.py`
```
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 10,
}
```

**Importação**
```
from rest_framework.pagination import PageNumberPagination
```

**Aplicação**
```
class RecipeAPI(ListCreateAPIView):
    queryset = Recipe.objects.get_published()
    serialzer_class = RecipeSerializer
    pagination_class = PageNumberPagination
```

## Personalização
```
class MyPaginationRecipe(PageNumberPagination):
    page_size = 12


class RecipeAPI(ListCreateAPIView):
    queryset = Recipe.objects.get_published()
    serialzer_class = RecipeSerializer
    pagination_class = MyPaginationRecipe
```