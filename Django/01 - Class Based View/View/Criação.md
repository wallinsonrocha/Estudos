# Django

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


```
from authors.forms.recipe_form import AuthorRecipeForm
from django.contrib import messages
from django.http.response import Http404
from django.shortcuts import redirect, render
from django.urls import reverse
from django.views import View
from recipes.models import Recipe


class DashboardRecipe(View):
    def get(self, request, id):
        recipe = Recipe.objects.filter(
            is_published=False,
            author=request.user,
            pk=id,
        ).first()

        if not recipe:
            raise Http404()

        form = AuthorRecipeForm(
            data=request.POST or None,
            files=request.FILES or None,
            instance=recipe
        )

        if form.is_valid():
            # Agora, o form é válido e eu posso tentar salvar
            recipe = form.save(commit=False)

            recipe.author = request.user
            recipe.preparation_steps_is_html = False
            recipe.is_published = False

            recipe.save()

            messages.success(request, 'Sua receita foi salva com sucesso!')
            return redirect(
                reverse('authors:dashboard_recipe_edit', args=(id,))
            )

        return render(
            request,
            'authors/pages/dashboard_recipe.html',
            context={
                'form': form
            }
        )
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
