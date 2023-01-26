# Rest Framework

[Documentação](https://www.django-rest-framework.org/api-guide/permissions/)

## Criando permissões personalizadas

Para isso recomenda-se criar um arquivo dentro da nossa aplicação para personalizar as permissões.

`permissions.py`

**Importação**
```
from rest_framework import permissions
```

**Classe**
```
from rest_framework import permissions

// Todos os dois métodos da classe podem ser modificados

class IsOwner(permissions.BasePermission):
    // Para checar um objeto
    def has_object_permission(self, request, view, obj):
        return obj.author == request.user

    // Para checar vários
    def has_permission(self, request, view):
        return super().has_permission(request, view)
```

## Implementação na view

Para o caso abaixo nós especificamos algumas autorizações.

```
...
from django.shortcuts import get_object_or_404
// Para permitir visualização ou autenticação
from rest_framework.permissions import IsAuthenticatedOrReadOnly
// Classe personalizada
from ..permissions import IsOwner
...
```
```
class RecipeAPIv2ViewSet(ModelViewSet):
    ...
    permission_classes = [IsAuthenticatedOrReadOnly, ]
```
```
    ...    
    def get_object(self):
        pk = self.kwargs.get('pk', '')

        obj = get_object_or_404(
            self.get_queryset(),
            pk=pk,
        )

        self.check_object_permissions(self.request, obj)

        return obj
    
    def get_permissions(self):
        if self.request.method in ['PATCH', 'DELETE']:
            return [IsOwner(), ]
        return super().get_permissions()

    def list(self, request, *args, **kwargs):
        print('REQUEST', request.user)
        print(request.user.is_authenticated)
        return super().list(request, *args, **kwargs)

    def partial_update(self, request, *args, **kwargs):
        recipe = self.get_object()
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