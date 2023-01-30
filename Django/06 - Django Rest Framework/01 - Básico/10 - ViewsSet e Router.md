# Rest Framework

[Documentação](https://www.django-rest-framework.org/api-guide/viewsets/)

## ModelViewSet
As Views set são muito utilizadas para simplificar a utilização dos métodos. Neste exemplo usaremos o ModelViewSet.

**Importação**
```
from rest_framework.viewsets import ModelViewSet
```

**Classe**
```
class RecipeAPIv2ViewSet(ModelViewSet):
    queryset = Recipe.objects.get_published()
    serializer_class = RecipeSerializer
    pagination_class = RecipeAPIv2Pagination

    def get_serializer_class(self):
        return super().get_serializer_class()

    def get_serializer(self, *args, **kwargs):
        return super().get_serializer(*args, **kwargs)

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context["example"] = 'this is in context now'
        return context

    def get_queryset(self):
        qs = super().get_queryset()

        category_id = self.request.query_params.get('category_id', '')

        if category_id != '' and category_id.isnumeric():
            qs = qs.filter(category_id=category_id)

        return qs

    def partial_update(self, request, *args, **kwargs):
        pk = kwargs.get('pk')

        recipe = self.get_queryset().filter(pk=pk).first()
        serializer = RecipeSerializer(
            instance=recipe,
            data=request.data,
            many=False,
            context={'request': request},
            partial=True,
        )
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(
            serializer.data,
        )
```

**Urls**
```
urlpatterns = [
    path(
        'recipes/api/v2/',
        views.RecipeAPIv2ViewSet.as_view({
            'get': 'list',
            'post': 'create',
        }),
        name='recipes_api_v2',
    ),
    path(
        'recipes/api/v2/<int:pk>/',
        views.RecipeAPIv2ViewSet.as_view({
            'get': 'retrieve',
            'patch': 'partial_update',
            'delete': 'destroy',
        }),
        name='recipes_api_v2_detail',
    ),
] 
```

## Router

Nós podemos simplificar a nomeação dos parâmetros com o simple router.

**Importação**
```
from rest_framework.routers import SimpleRouter
```

**Urls**
```
// Configuração
recipe_api_v2_router = SimpleRouter()
recipe_api_v2_router.register(
    'recipes/api/v2',
    views.RecipeAPIv2ViewSet,
    basename='recipes-api',
)

urlpatterns = [
    ...
    path('', include(recipe_api_v2_router.urls)),
]
```

Dessa forma não seria mais necessário especificar os métodos e as funções em nossa ViewSet.

[Outro exemplo](https://github.com/luizomf/curso-django-projeto1/blob/561dfebffad5bb7df6700a1659b97ec27d4efa14/recipes/views/api.py)