# Rest Framework

As validações serão feitas na classe que está o serializer. Se queremos criar uma validação para RecipeSerializer, por exemplo, devemos criar suas validações lá.

Assim como o form que possui o clean, no serializer é o `validate` ou `validate_campo`

```
def validate(self, attrs):
        super_validate = super().validate(attrs)

        title = attrs.get('title')
        description = attrs.get('description')

        if title == description:
            raise serializers.ValidationError(
                {
                    "title": ["Posso", "ter", "mais de um erro"],
                    "description": ["Posso", "ter", "mais de um erro"],
                }
            )

        return super_validate

def validate_title(self, value):
    title = value

    if len(title) < 5:
        raise serializers.ValidationError('Must have at least 5 chars.')

    return title
```