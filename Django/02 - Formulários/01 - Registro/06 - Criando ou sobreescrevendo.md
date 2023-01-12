# Django

## Criar ou sobreescrever

Podemos fazer tudo isso através do **fomrs.CharField**. Dentro dele podemos colocar alguns parâmetros. Observe os exemplos abaixo.

```
password = forms.CharField(
    required=True,
    widget=forms.PasswordInput(attrs={
        'placeholder': 'Your password'
    }),
    error_messages={
        'required': 'Password must not be empty'
    },
    help_text=(
        'Password must have at least one uppercase letter, '
        'one lowercase letter and one number. The length should be '
        'at least 8 characters.'
    )
)

password2 = forms.CharField(
    required=True,
    widget=forms.PasswordInput(attrs={
        'placeholder': 'Repeat your password'
    })
)
```