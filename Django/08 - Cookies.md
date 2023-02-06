# Cookies

[Documentação](https://docs.djangoproject.com/pt-br/4.1/topics/http/sessions/)

Usado para guardar dados não sensíveis e de menor importância.

Ao configurar um cookie devemos levar em consideração que ele têm alguns parâmetros principais.
- `key` - Obrigatório
- `value` - Obrigatório
- `max_age` - Inteiro em segundos
- `domain` - exemple.com
- `secure` - True apenas https
- `httponly` - True para JS não acessar

## Set_cookie
Para criar um novo ccokie podemos usar a função `set_cookie`:
```
def home(request):
    response = HttpResponse()
    response.set_cookie('name', value)
    return response
```

## Get
```
def home(request):
    cookies = request.COOKIES.get('name')
```

