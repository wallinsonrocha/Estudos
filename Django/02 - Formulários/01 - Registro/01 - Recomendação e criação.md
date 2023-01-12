# Django

## Preparação
Recomenda-se criar um novo app chamado authors para personalizar as permissões e os tipos de usuários separadamente.
```
python manage.py startapp authors
```

Após isso basta seguirmos as configurações padrões para configurar o nosso app ([ver aqui]()), seguiremos as seguintes recomendações:

### Em `urls.py` do projeto iremos reconfigurá-lo dessa forma:
```
urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('recipes.urls')),
    path('authors/', include('authors.urls')),
]
```

Para author iremos colocar um caminho padrão antes de fazer qualquer direcionamento. Ou seja será `site.com/authors/alguma_coisa/`.

### View

Modelo de view

```
from .forms import RegisterForm
...

def register_view(request):
    register_form_data = request.session.get('register_form_data', None)
    form = RegisterForm(register_form_data)
    return render(request, 'authors/pages/register_view.html', {
        'form': form,
        'form_action': reverse('authors:register_create'),
    })


def register_create(request):
    if not request.POST:
        raise Http404()

    POST = request.POST
    request.session['register_form_data'] = POST
    form = RegisterForm(POST)

    if form.is_valid():
        user = form.save(commit=False)
        user.set_password(user.password)
        user.save()
        messages.success(request, 'Your user is created, please log in.')

        del(request.session['register_form_data'])

    return redirect('authors:register')
```

## URL da aplicação
```
from . import views
...

urlpatterns = [
    path('register/', views.register_view, name='register'),
    path('register/create/', views.register_create, name='register_create'),
]
```