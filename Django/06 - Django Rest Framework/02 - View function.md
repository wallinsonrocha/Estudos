# Rest Framework

```
# Views

@api_view()
def recipe_api_detail(request, pk):
    recipe = get_object_or_404(
        Recipe.objects.get_published(),
        pk=pk
    )
    serializer = RecipeSerializer(instance=recipe, many=False)
    return Response(serializer.data)

# Urls app

path(
        'recipes/api/v2/<int:pk>/',
        views.recipe_api_detail,
        name='recipes_api_v2_detail',
    )
```