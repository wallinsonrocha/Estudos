# Rest Framework

## Permidinto alguns métodos
Caso desejamos selecionar apenas alguns métodos, podemos por em um atributo da nossa classe para que alguns métodos sejam permitidos.
```
class RecipeAPIv2ViewSet(ModelViewSet):    
    http_method_names = ['get', 'options', 'head', 'patch', 'post', 'delete']
```

Nesse caso o método PUT não será permitido.