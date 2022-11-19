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

Caso deixarmos o field assim, ele irá mostrar todas as opções. Mas, para limitarmos alguns acessos ao usuário, podemos escrever assim:
```
    fields = [
        'first_name',
        'last_name',
        'username',
        'email',
        'password',
    ]
```

## Colocando-o na view
```
from .forms import RegisterForm
...

def register_view(request):
    form = RegisterForm()
    return render(request, 'authors/pages/register_view.html',{
        'form': form,
    })
```