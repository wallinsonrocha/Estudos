## 1️⃣ Instalação

Usaremos o pacote oficial **djangorestframework-simplejwt**:

```bash
pip install djangorestframework-simplejwt
```

---

## 2️⃣ Configuração do Django REST Framework

```python
# settings.py
INSTALLED_APPS = [
    # ... apps do Django
    'rest_framework',
    'rest_framework_simplejwt.token_blacklist',  # necessário para blacklist de refresh tokens
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}
```

---

## 3️⃣ URLs para JWT

```python
# urls.py
from django.urls import path
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from .views import CustomTokenRefreshView

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', CustomTokenRefreshView.as_view(), name='token_refresh'),
]
```

> A ideia é substituir o endpoint `TokenRefreshView` padrão por um customizado que **envia o refresh token via HttpOnly cookie**, não no body.

---

## 4️⃣ Criar login que retorna refresh token HttpOnly

```python
# views.py
from rest_framework_simplejwt.tokens import RefreshToken
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.contrib.auth import authenticate

class LoginView(APIView):
    def post(self, request):
        username = request.data.get("username")
        password = request.data.get("password")
        user = authenticate(username=username, password=password)

        if user is not None:
            refresh = RefreshToken.for_user(user)
            access_token = str(refresh.access_token)

            # Configura refresh token como HttpOnly cookie
            response = Response({"access": access_token})
            response.set_cookie(
                key='refresh_token',
                value=str(refresh),
                httponly=True,
                secure=True,          # usar True se estiver com HTTPS
                samesite='Strict',    # ou 'Lax' dependendo da necessidade
                max_age=7*24*60*60    # 7 dias
            )
            return response
        else:
            return Response({"error": "Credenciais inválidas"}, status=status.HTTP_401_UNAUTHORIZED)
```

✅ Aqui:

* O **access token** vai no body da resposta (JSON).
* O **refresh token** vai no **HttpOnly cookie**, invisível para JS.

---

## 5️⃣ Endpoint de refresh usando HttpOnly cookie

```python
# views.py
from rest_framework_simplejwt.views import TokenRefreshView
from rest_framework_simplejwt.exceptions import InvalidToken, TokenError

class CustomTokenRefreshView(TokenRefreshView):
    def post(self, request, *args, **kwargs):
        refresh_token = request.COOKIES.get('refresh_token')

        if refresh_token is None:
            return Response({"error": "Refresh token não encontrado"}, status=401)

        serializer = self.get_serializer(data={'refresh': refresh_token})

        try:
            serializer.is_valid(raise_exception=True)
        except TokenError as e:
            raise InvalidToken(e.args[0])

        # retorna novo access token
        access_token = serializer.validated_data['access']
        return Response({'access': access_token})
```

---

## 6️⃣ Logout (opcional, para apagar cookie)

```python
# views.py
from rest_framework.permissions import IsAuthenticated

class LogoutView(APIView):
    permission_classes = [IsAuthenticated]

    def post(self, request):
        response = Response({"message": "Logout realizado"})
        response.delete_cookie('refresh_token')
        return response
```

---

## 7️⃣ Fluxo resumido

1. Cliente faz `POST /login/` com usuário e senha
   → Recebe `access token` no body e `refresh token` no HttpOnly cookie.
2. Cliente envia `Authorization: Bearer <access>` nas requisições protegidas
3. Quando access expirar, cliente faz `POST /api/token/refresh/` sem body
   → O servidor lê o refresh token do HttpOnly cookie e retorna **novo access token**
4. Logout apaga o cookie.

---

💡 **Dicas práticas:**

* Sempre use `secure=True` em produção (HTTPS obrigatório).
* `samesite='Strict'` ajuda a prevenir CSRF.
* O access token é curto (ex: 5 min), o refresh token dura dias.
* Evite armazenar tokens em localStorage ou sessionStorage: HttpOnly é mais seguro.

---

