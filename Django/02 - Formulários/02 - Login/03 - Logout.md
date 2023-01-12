# Django

## View

```
from django.contrib.auth.decorators import login_required
...

# Para realizar apenas se o usuário estiver logado.
@login_required(login_url='authors:login', redirect_field_name='next')
def logout_view(request):
    if not request.POST:
        return redirect(reverse('authors:login'))

    if request.POST.get('username') != request.user.username:
        return redirect(reverse('authors:login'))

    logout(request)
    return redirect(reverse('authors:login'))
```

## Url da aplicação
```
urlpatterns = [
    path('logout/', views.logout_view, name='logout'),
]
```

## Modelo para página
```
 <div class="main-content center container">
    <h2>Login</h2>
    # Importante. O usuário deve estar logado
    {% if request.user.is_authenticated %}
      <div>
        Your are logged in with 
        {{ request.user.username }}. 
        Please, 



        # Parte para logout
        <form class="inline-form" action="{% url 'authors:logout' %}" method='POST'>
          {% csrf_token %}
          <input type="hidden" name="username" value="{{ request.user.username }}">
          # Importane. Ele que irá direcionar para o logout.
          <button class="plaintext-button" type="submit">click here</button>
        </form> to logout.



      </div>
    {% endif %}
  </div>
```