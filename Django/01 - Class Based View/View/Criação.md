# Django

Uma Class-Based View (CBV) é uma classe do Python que define a lógica de uma view em uma aplicação Django. A View é a classe base de todas as CBVs no Django e oferece uma maneira simples de lidar com diferentes tipos de requisições HTTP em uma única classe.

A estrutura de uma View consiste em 5 métodos principais:

- `dispatch`: É o primeiro método chamado para todas as requisições e é responsável por redirecionar a requisição para um dos métodos abaixo, dependendo do tipo de requisição recebida.
- `get`: Lida com requisições GET.
- `post`: Lida com requisições POST.
- `put`: Lida com requisições PUT.
- `delete`: Lida com requisições DELETE.

O método `dispatch` é um método interno da View e é chamado automaticamente pelo framework do Django para cada requisição que é feita à view. Ele é responsável por determinar o tipo de requisição e redirecionar a requisição para o método correspondente (`get`, `post`, `put`, `delete`). Caso a requisição não seja de um tipo suportado, o método `dispatch` retorna um erro HTTP 405 Method Not Allowed.

Cada um dos métodos acima (`get`, `post`, `put`, `delete`) é responsável por lidar com as respectivas requisições HTTP e deve retornar uma resposta HTTP apropriada. Por exemplo, o método `get` pode retornar uma resposta HTTP que contém um modelo ou uma lista de modelos (em formato HTML, JSON, XML etc.). O método `post` pode ser usado para salvar um novo modelo na base de dados.

Além desses métodos principais, uma CBV também pode ter outros métodos personalizados que são usados para realizar outras tarefas específicas.


## Funções da class based view View

A `View` é uma classe base do Django que não fornece nenhuma funcionalidade específica, mas é útil como uma classe base para outras views. Aqui estão algumas das funções da `View`:

- `dispatch`: é o método que chama o método correto da view com base no método HTTP da requisição. Por padrão, é chamado um dos métodos `get`, `post`, `put`, `delete`, `head`, `options` ou `trace`, dependendo do método HTTP. Este método geralmente não precisa ser sobrescrito, a menos que você esteja criando um comportamento especializado.

- `http_method_not_allowed`: é chamado se a requisição usa um método HTTP não permitido para a view (por exemplo, uma requisição `POST` para uma view que não permite `POST`). Por padrão, ele retorna uma resposta `HttpResponseNotAllowed`, mas pode ser sobrescrito para personalizar o comportamento.

- `options`: é o método chamado para lidar com uma requisição `OPTIONS` na view. Por padrão, ele retorna uma resposta com um cabeçalho `Allow` contendo os métodos HTTP permitidos para a view.

- `as_view`: é um método estático que retorna uma função que pode ser chamada com uma instância da classe para processar uma requisição. É usado para conectar a view a uma URL. Por exemplo, `url(r'^myurl/$', MyView.as_view(), name='myview')`.

- `setup`: é o método que é chamado quando a view é inicializada. Pode ser usado para inicializar coisas como variáveis de classe ou objetos de banco de dados.

- `get`: é o método que lida com requisições `GET`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `GET`.

- `post`: é o método que lida com requisições `POST`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `POST`.

- `put`: é o método que lida com requisições `PUT`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `PUT`.

- `delete`: é o método que lida com requisições `DELETE`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `DELETE`.

- `head`: é o método que lida com requisições `HEAD`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `HEAD`.

- `options`: é o método que lida com requisições `OPTIONS`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `OPTIONS`.

- `trace`: é o método que lida com requisições `TRACE`. É aqui que você deve colocar a maior parte do seu código para lidar com requisições `TRACE`.


## Estrutura de uma Class-Based View (CBV) View e suas funções

```python
from django.views.generic import View

class MinhaView(View):

    def dispatch(self, request, *args, **kwargs):
        # Método que é chamado para redirecionar a view para os outros métodos abaixo, dependendo do tipo de request
        if request.method == 'GET':
            return self.get(request, *args, **kwargs)
        elif request.method == 'POST':
            return self.post(request, *args, **kwargs)
        elif request.method == 'PUT':
            return self.put(request, *args, **kwargs)
        elif request.method == 'DELETE':
            return self.delete(request, *args, **kwargs)
        else:
            # Retorna um erro 405 Method Not Allowed caso o método HTTP não seja suportado
            return HttpResponseNotAllowed(['GET', 'POST', 'PUT', 'DELETE'])
            
    def get(self, request, *args, **kwargs):
        # Método que lida com requisições GET
        return HttpResponse('Requisição GET recebida!')
        
    def post(self, request, *args, **kwargs):
        # Método que lida com requisições POST
        return HttpResponse('Requisição POST recebida!')
        
    def put(self, request, *args, **kwargs):
        # Método que lida com requisições PUT
        return HttpResponse('Requisição PUT recebida!')
        
    def delete(self, request, *args, **kwargs):
        # Método que lida com requisições DELETE
        return HttpResponse('Requisição DELETE recebida!')
```

## Urls
No nosso arquivo `urls.py` faremos algumas alterações.
```
urlpatterns = [
    path(
        'dashboard/recipe/<int:id>/edit/',
        views.DashboardRecipe.as_view(),
        name='dashboard_recipe_edit'
    ),
]
```

Para que a importação seja reconhecida devemos importar dessa forma: `views.Class.as_view()`.

## Caso haja mais de uma classe
Para resolver isso basta seguir o mesmo processo feito para os testes. Criamos um pacote que contenha o arquivo `__init__.py` e importar as classes criadas.
