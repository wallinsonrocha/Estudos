# Django

## Validar campo específico

A validação é feita juntamente com a classe criadora do formulário que estamos usando.

Devemos colocar o nome do campo depois de **clean_**. Observe o exemplo abaixo. Basta colocar a função dentro da classe que ela será executada automaticamente.

```
from django.core.exceptions import ValidationError
...

def clean_password(self):
    # Coletar dado
    data = self.cleaned_data.get('password') 

    # Validação
    if 'atenção' in data:
        raise ValidationError(
            'Valor inválido', #mensagem
            code='invalid',
            params={value: 'atenção'}
        )

    # Caso validação OK
    return data

```

## Validar campo dependente de outro


```
from django.core.exceptions import ValidationError
...

 def clean(self):
        cleaned_data = super().clean()

        password = cleaned_data.get('password')
        password2 = cleaned_data.get('password2')

        if password != password2:
            password_confirmation_error = ValidationError(
                'Password and password2 must be equal',
                code='invalid'
            )

            raise ValidationError({
                'password': password_confirmation_error,
                # Uma lista caso haja mais de um erro.
                'password2': [
                    password_confirmation_error,
                ],
            })
```

## Verificação de email único

É de suma importância checar se se o email já é utilizado.
```
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError
...

def clean_email(self):
    email = self.cleaned_data.get('email', '')
    exists = User.objects.filter(email=email).exists()

    if exists:
        raise ValidationError(
            'User e-mail is already in use', code='invalid',
        )

    return email
```