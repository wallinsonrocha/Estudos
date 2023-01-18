# Rest Framework

```
from rest_framework import serializers


class TagSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    name = serializers.CharField(max_length=255)
    slug = serializers.SlugField()



class RecipeSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    title = serializers.CharField(max_length=65)
    description = serializers.CharField(max_length=165)

    # renomear atributos
    public = serializers.BooleanField(source='is_published') 

    # Juntar atributos
    preparation = serializers.SerializerMethodField(
        method_name='any_method_name'
    )

    # Retorna o nome do objeto referenciado. Neste caso, many no one
    category = serializers.StringRelatedField()

    # Retorna o ID do objeto referenciado
    author = serializers.PrimaryKeyRelatedField(
        queryset=User.objects.all()
    )

    # Para retornar os IDs de um many to many
    tags = serializers.PrimaryKeyRelatedField(
        queryset=Tag.objects.all(),
        many=True
    )

    # Colocando a referência de um outro model Serializer, podemos retornar tods os atributos que queremos dessa classe relacionada.
    tag_objects = TagSerializer(
        many=True, source='tags'
    )

    def any_method_name(self, recipe):
        return f'{recipe.preparation_time} {recipe.preparation_time_unit}'
```

## Atributo read_only
Além disso, podemos colocar o atributo read_only para dizer se tal atributo é ou não editável.

```
tag_links = serializers.HyperlinkedRelatedField(
        many=True,
        source='tags',
        view_name='recipes:recipes_api_v2_tag',
        read_only=True,
    )
```

## Para HtperLinkedRelatedField

```
# serializers
...
    tag_links = serializers.HyperlinkedRelatedField(
        many=True,
        source='tags',
        queryset=Tag.objects.all(),
        view_name='recipes:recipes_api_v2_tag'
    )
...

# urls
 path(
        'recipes/api/v2/tag/<int:pk>/',
        views.tag_api_detail,
        name='recipes_api_v2_tag',
    ),

# view
@api_view()
def recipe_api_list(request):
    recipes = Recipe.objects.get_published()[:10]
    serializer = RecipeSerializer(
        instance=recipes,
        many=True,
        context={'request': request},
    )
    return Response(serializer.data)
```