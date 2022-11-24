# Django

## View
Renderizar página para Login que criaremos.
```
from .forms import LoginForm
...

def login_view(request):
    form = LoginForm()
    return render(request, 'authors/pages/login.html', {
        'form': form,
        'form_action': reverse('authors:login_create')
    })


def login_create(request):
    return render(request, 'authors/pages/login.html')
```

## URL da aplicação
```
urlpatterns = [
    path('login/', views.login_view, name='login'),
    path('login/create/', views.login_create, name='login_create'),
]
```