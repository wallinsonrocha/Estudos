# Django

Todo o código é feito fora da classe principal. Ou seja, é feito antes de tudo.

## Função para adicionar
Fora da classe Meta nós podemos sobreescrever alguns inputs. Para isso, podemos colocar algumas funções para agilizar o processo;

### add_attr

Para sobrescrever ou criar atributos
```
def add_attr(field, attr_name, attr_new_val):
    existing = field.widget.attrs.get(attr_name, '')
    field.widget.attrs[attr_name] = f'{existing} {attr_new_val}'.strip()
```

### add_placeholder

Para simplificar a sobreescrita no placeholder
```
def add_placeholder(field, placeholder_val):
    add_attr(field, 'placeholder', placeholder_val)
```

## Utilização
```
def __init__(self, *args, **kwargs):
    super().__init__(*args, **kwargs)
    add_placeholder(self.fields['username'], 'Your username')
    add_placeholder(self.fields['email'], 'Your e-mail')
    add_placeholder(self.fields['first_name'], 'Ex.: John')
    add_placeholder(self.fields['last_name'], 'Ex.: Doe')
    add_attr(self.fields['username'], 'css', 'a-css-class')
```

## Arquivo alternativo

Podemos criar um arquivo próprio para funções uma vez que podemos usar em outras forms. Podemos criá-lo dentro da pasta utils onde está o pagination.

```
import re
from django.core.exceptions import ValidationError


def add_attr(field, attr_name, attr_new_val):
    existing = field.widget.attrs.get(attr_name, '')
    field.widget.attrs[attr_name] = f'{existing} {attr_new_val}'.strip()


def add_placeholder(field, placeholder_val):
    add_attr(field, 'placeholder', placeholder_val)


def strong_password(password):
    regex = re.compile(r'^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9]).{8,}$')

    if not regex.match(password):
        raise ValidationError((
            'Password must have at least one uppercase letter, '
            'one lowercase letter and one number. The length should be '
            'at least 8 characters.'
        ),
            code='invalid'
        )
```

Após isso, basta importarmos as nossas funções no arquivo que estamos usando.