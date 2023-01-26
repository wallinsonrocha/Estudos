# Rest Framework

[Documentação](https://www.django-rest-framework.org/api-guide/generic-views/)

## Class Based View

**Importação**
```
from rest_framework.views import APIView
from rest_framework.views import ListCreateAPIView
```

### APIView
```
class RecipeAPIv2List(APIView):
    def get(self, request):
        recipes = Recipe.objects.get_published()[:10]
        serializer = RecipeSerializer(
            instance=recipes,
            many=True,
            context={'request': request},
        )
        return Response(serializer.data)

    def post(self, request):
        serializer = RecipeSerializer(
            data=request.data,
            context={'request': request},
        )
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(
            serializer.data,
            status=status.HTTP_201_CREATED
        )
```

### ListCreateAPIView
```
class RecipeAPI(ListCreateAPIView):
    queryset = Recipe.objects.get_published()
    serialzer_class = RecipeSerializer
```