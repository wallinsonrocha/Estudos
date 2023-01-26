# Rest Framework

## Métodos

Nós podemos configurar a nossa view para receber respostas. Podemos por como get, post, delete, update e outros.
```
@api_view(http_method_names=['get', 'post'])
def recipe_api_list(request):
    if request.method == 'GET':
        recipes = Recipe.objects.get_published()[:10]
        serializer = RecipeSerializer(  
            #instance = nome do app (namespace)
            instance=recipes,
            many=True,
            context={'request': request},
        )
        return Response(serializer.data)


    elif request.method == 'POST':
        serializer = RecipeSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        serializer.save()
        return Response(
            serializer.validated_data,
            status=status.HTTP_201_CREATED
        )
```