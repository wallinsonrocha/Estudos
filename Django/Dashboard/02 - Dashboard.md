# Django

## View
```
@login_required(login_url='authors:login', redirect_field_name='next')
def dashboard(request):
    return render(request, 'authors/pages/dashboard.html')
    # Responsável pela exposição dos conteúdos
    recipes = Recipe.objects.filter(
        is_published=False,
        author=request.user
    )
    return render(request, 'authors/pages/dashboard.html',
        context={
            'recipes': recipes,
        }
    )
```
> Class.objects é responsável por fazer essa exposição de acordo com a nossa configuração. 
> É de suma importância, nesse caso, o login estar ativo.

```
@login_required(login_url='authors:login', redirect_field_name='next')
def dashboard_recipe_edit(request, id):
    # Aquilo que queremos permitir que o usuário acesse das receitas
    recipe = Recipe.objects.filter(
        is_published=False,
        author=request.user,
        pk=id,
    ).first()

    if not recipe:
        raise Http404()

    form = AuthorRecipeForm(
        data=request.POST or None,
        # Caso em nosso form venha um arquivo
        files=request.FILES or None,
        instance=recipe
    )

    return render(
        request,
        'authors/pages/dashboard_recipe.html',
        context={
            'form': form
        }
    )
```

> Quando em nosso form tem algum arquivo que irá ser enviado, devemos colocar isso em nossa tag.
```
<form 
    class="main-form"
    action="{{ form_action }}" 
    method="POST"
    enctype="multipart/form-data" # Este é o principal responsável para indicar que há arquivos
  >
```

## Form para criar
```
from collections import defaultdict

from django import forms
from django.core.exceptions import ValidationError
from recipes.models import Recipe
from utils.django_forms import add_attr
from utils.strings import is_positive_number


class AuthorRecipeForm(forms.ModelForm):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        self._my_errors = defaultdict(list)

        add_attr(self.fields.get('preparation_steps'), 'class', 'span-2')

    class Meta:
        model = Recipe
        fields = 'title', 'description', 'preparation_time', \
            'preparation_time_unit', 'servings', 'servings_unit', \
            'preparation_steps', 'cover'
        widgets = {
            'cover': forms.FileInput(
                attrs={
                    'class': 'span-2'
                }
            ),
            'servings_unit': forms.Select(
                choices=(
                    ('Porções', 'Porções'),
                    ('Pedaços', 'Pedaços'),
                    ('Pessoas', 'Pessoas'),
                ),
            ),
            'preparation_time_unit': forms.Select(
                choices=(
                    ('Minutos', 'Minutos'),
                    ('Horas', 'Horas'),
                ),
            ),
        }

    def clean(self, *args, **kwargs):
        super_clean = super().clean(*args, **kwargs)
        cd = self.cleaned_data

        title = cd.get('title')
        description = cd.get('description')

        if title == description:
            self._my_errors['title'].append('Cannot be equal to description')
            self._my_errors['description'].append('Cannot be equal to title')

        if self._my_errors:
            raise ValidationError(self._my_errors)

        return super_clean

    def clean_title(self):
        title = self.cleaned_data.get('title')

        if len(title) < 5:
            self._my_errors['title'].append('Must have at least 5 chars.')

        return title

    def clean_preparation_time(self):
        field_name = 'preparation_time'
        field_value = self.cleaned_data.get(field_name)

        if not is_positive_number(field_value):
            self._my_errors[field_name].append('Must be a positive number')

        return field_value

    def clean_servings(self):
        field_name = 'servings'
        field_value = self.cleaned_data.get(field_name)

        if not is_positive_number(field_value):
            self._my_errors[field_name].append('Must be a positive number')

        return field_value
```