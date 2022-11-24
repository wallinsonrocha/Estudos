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