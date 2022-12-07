# Django

## Class Based View 
As class based views é uma forma alternativa de construir views para a nossa aplicação. Supomos que fora criada um arquivo chamado `dashboard_recipe.py`.

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