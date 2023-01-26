# Rest Framework

Para criar uma url nova podemos utilizar um decoration.

**Importação**
```
from rest_framework.decorators import action
```

**Implementação na view**
```
def get_queryset(self):
        User = get_user_model()
        qs = User.objects.filter(username=self.request.user.username)
        return qs

    @action(
        methods=['get'],
        detail=False,
    )
    def me(self, request, *args, **kwargs):
        obj = self.get_queryset().first()
        serializer = self.get_serializer(
            instance=obj
        )
        return Response(serializer.data)
```