# Django

## 1 - Testar existencia da url
`ClassTest`
```
    def test_projectsPort_home_url_is_correct(self):
        url = reverse('projetos_port:home')
        self.assertEqual(url, '/')
```
Usaremos o `reverse` para fazer a referencia da url.


`Urls.py`
```
    path('', lambda request: ..., name = "home"),
```

## 2 - Criar e testar view para implementar na url (Com class Based View)
`View`
```
from django.views.generic import View

# Create your views here.

class ProjectsPortViewBase(View):
    ...


class ProjectsPortViewHome(ProjectsPortViewBase):
    template_name = ...
```


`ClassTest`
```
    def test_projectsPort_home_view_is_correct(self):
        resolved = resolve(reverse('projetos_port:home'))
        self.assertIs(resolved.func.view_class, views.ProjectsPortViewHome)
```

Usaremos o `resolve` para fazer a referência da view pela url.

## 3 - Carregando o template correto
`View`
```
template_name = 'projetos_port/pages/home.html'

class ProjectsPortViewBase(View):
    def get(self, request):
        return render(
                request,
                template_name,
            )


class ProjectsPortViewHome(ProjectsPortViewBase):
    ...   
```

`ClassTest`
```
    def test_projectsPort_load_correct_template_home(self):
        response = self.client.get(reverse('projetos_port:home'))
        self.assertTemplateUsed(response, 'projetos_port/pages/home.html')
```
