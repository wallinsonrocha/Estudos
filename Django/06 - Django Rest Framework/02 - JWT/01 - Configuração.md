# Rest framework

[Documentação](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/getting_started.html)

## Simple JWT configurações

**Instalação**
```
pip install djangorestframework-simplejwt
```

**Implementação na pasta `/settings` ou  `settings.py`**
Dentro da pasta settings iremos implementar algumas linhas de consiguração. Por isso, preferencialmente, criaremos um arquivo para colocar algumas configurações. Podemos chamar de `rest_framework.py`.
```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
}
```

E

```
from datetime import timedelta
...

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=5), // Para testes esse tempo pode ser maior
    'REFRESH_TOKEN_LIFETIME': timedelta(days=1),
    'BLACKLIST_AFTER_ROTATION': False,
    'SIGNING_KEY': 'SECRET_KEY', // Personalizada no .env
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

Podemos colocar, também, algumas configurações como a do django pagination. Só não podemos esquecer de importá-lo no `__init__.py`
```
from .rest_framework import *
```

**Importação**

A importação pode ser feita na url root da nossa aplicação ou nas urls das aplicações.
```
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView
)

...

urlpatterns = [
    ...
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
    ...
]
```

**Para traduções ou localizações**
```
INSTALLED_APPS = [
    ...
    'rest_framework_simplejwt',
    ...
]
```