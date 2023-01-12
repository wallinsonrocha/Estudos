# Django

## Criando POST na view
Quando fazemos o direcionamento de um formulário devemos enviá-lo em um local para receber esses dados. Então, para isso, podemos criar uma função para isso na nossa `view.py`.

```
from django.http import Http404
from django.shortcuts import redirect, render

...
from django.contrib import messages
...

def register_create(request):
    if not request.POST: # Para aceitar apenas POST
        raise Http404()

    POST = request.POST
    request.session['register_form_data'] = POST
    form = RegisterForm(POST)

    # Validação caso o form seja válido
    if form.is_valid():
        # Armazenado para modificação
        user = form.save(commit=False)
        # Criptografando senha
        user.set_password(user.password)
        user.save()
        messages.success(request, 'Your user is created, please log in.')

        del(request.session['register_form_data'])

    return redirect('authors:register')
```

Observe que ele retorna um redirecionamento. Esse direcionamento é para o register.

Há uma mútua conexão entre o **register_view** e o **register_create**.

Ele só irá entrar no **register_create** se clicarmos no botão do tipo submit. Ou seja, se enviarmos o formulário para a url se criação. (Vemos como fazer isso em "Implementando em nossa action")

## Direcionando-o para URLs
Como criamos um direcionamento novo, precisamos implementá-lo em nossa `urls.py` da aplicação.
```
from . import views
...

urlpatterns = [
    path('register/', views.register_view, name='register'),
    path('register/create/', views.register_create, name='register_create'),
]
```

## Implementando em nossa action
Para implementar devemos colocar na action da nossa form no arquivo html.
```
<form action="{% url 'authors:register_create' %}" method="POST">
    ...
</form>
```

Dessa forma `{% url 'authors:register_create' %}` é a template e o nome da url que criamos em `urls.py`.