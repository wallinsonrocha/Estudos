# Django

## Criando arquivo
Primeiramente vamos criar um arquivo chamado `forms.py` dentro da pasta da nossa aplicação. Colocaremos:
```
from django import forms
from django.contrib.auth.models import User

class RegisterForm(forms.ModelForm):
    class Meta:
        model = User
        fields = '__all__'
```

> Não podemos esquecer de criar a Meta classe para colocar os campos.

Caso deixarmos o field assim, ele irá mostrar todas as opções. Mas, para limitarmos alguns acessos ao usuário, podemos escrever assim:
```
    fields = [
        'first_name',
        'last_name',
        'username',
        'email',
        'password',
    ]

    labels = {
        'first_name' : 'First name'
        'last_name' : 'Last name'
        'username' : 'Username'
        'email' : 'Email'
        'password' : 'Password'    
    }

    help_texts = {
        'email' = 'The e-mal must be valid.',
    }
```

> Dentro do label ficará a representação de como nós queremos que cada label seja nomeado.
> Além disso há o help_texts.

## Colocando-o na view
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
```

A **request.session** é a sessão do usuário. Ajuda a fazer comunicação entre requisições.

