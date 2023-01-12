# Django

## View
Renderizar página para Login que criaremos.
```
from .forms import LoginForm
from django.contrib.auth import authenticate, login
...

def login_view(request):
    form = LoginForm()
    return render(request, 'authors/pages/login.html', {
        'form': form,
        'form_action': reverse('authors:login_create')
    })


def login_create(request):
    if not request.POST:
        raise Http404()

    form = LoginForm(request.POST)
    login_url = reverse('authors:login')

    #Verificar se o formulário é válido
    if form.is_valid():
        authenticated_user = authenticate(
            username=form.cleaned_data.get('username', ''),
            password=form.cleaned_data.get('password', ''),
        )

        # Verificar se é possível fazer autenticação com as credenciais utilizadas.
        if authenticated_user is not None:
            messages.success(request, 'Your are logged in.')
            login(request, authenticated_user)
        else:
            messages.error(request, 'Invalid credentials')
    else:
        messages.error(request, 'Invalid username or password')

    return redirect(login_url)
```

## URL da aplicação
```
urlpatterns = [
    path('login/', views.login_view, name='login'),
    path('login/create/', views.login_create, name='login_create'),
]
```

## Modelo de página
```
  <div class="main-content center container">
    <h2>Login</h2>
    {% if request.user.is_authenticated %}
      <p>
        Your are logged in with 
        {{ request.user.username }}.
      </p>
    {% endif %}
  </div>
```

