# Rest Framework

## Respostas

Ao fazer algumas requisições teremos algumas respostas.

### api/token/
Através do método post poderemos enviar username e password. Teremos o retorno de um access e um refresh.
```
// Enviar
{
    "username": "name",
    "password": "pass"
}

// Resposta
{
    "refresh": "ljahdshsagdiohsadljkh",
    "access" : "ljashdjhsadkhsaldhsjad"
}
```

Esses dados podem ser salvos no local storage ou outros lugares.

O refresh irá entregar novos access (que possuem durações curtas) a fim de permanecer o usuário logado.

### api/token/refresh/

Para retornar um novo access token e valida o refresh token.
```
// Enviar
{
    "refresh": "ljahdshsagdiohsadljkh"
}

// Resposta
{
    "access": "oakjsdkljsdkjaskjdla"
}
```

### api/token/verify/

Para verificar o refresh.
```
{
    "token": "lskasçlalçskdçlask"
}
```