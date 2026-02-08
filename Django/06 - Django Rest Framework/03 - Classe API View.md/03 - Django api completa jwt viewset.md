# 🔐 API Completa: ViewSet + JWT + Permissões (Exemplo Real)

## 🧱 Cenário

Vamos construir uma API REST completa e **production-ready** com:

* ✅ **Produto** → modelo simples mas completo
* ✅ **Admin** → CRUD completo (criar, ler, atualizar, deletar)
* ✅ **Usuário comum** → apenas leitura (GET)
* ✅ **JWT** com access token + refresh token
* ✅ **Paginação** customizada com mensagens claras
* ✅ **Filtros, busca e ordenação**
* ✅ **Mensagens customizadas** em todas as respostas
* ✅ **Validações** robustas
* ✅ **Testes** automatizados

---

## 📦 1. Instalação e Dependências

```bash
pip install djangorestframework
pip install djangorestframework-simplejwt
pip install django-filter
```

```python
# settings.py - INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # DRF
    'rest_framework',
    'rest_framework_simplejwt',
    'rest_framework_simplejwt.token_blacklist',  # Para invalidar tokens
    'django_filters',
    
    # Sua app
    'products',
]
```

---

## 🔧 2. Configuração do JWT

```python
# settings.py
from datetime import timedelta

# Configuração do Django REST Framework
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}

# Configuração do SimpleJWT
SIMPLE_JWT = {
    # Tempo de vida dos tokens
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    
    # Rotacionar refresh token a cada uso
    'ROTATE_REFRESH_TOKENS': True,
    
    # Blacklist do token antigo após rotação
    'BLACKLIST_AFTER_ROTATION': True,
    
    # Atualizar last_login do usuário
    'UPDATE_LAST_LOGIN': True,
    
    # Algoritmo de criptografia
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    
    # Header da requisição
    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
    
    # Claims do token
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
    
    # Configurações adicionais
    'AUTH_TOKEN_CLASSES': ('rest_framework_simplejwt.tokens.AccessToken',),
    'TOKEN_TYPE_CLAIM': 'token_type',
}
```

---

## 🗄️ 3. Modelo

```python
# models.py
from django.db import models
from django.core.validators import MinValueValidator
from decimal import Decimal

class Product(models.Model):
    name = models.CharField(
        max_length=120,
        verbose_name='Nome do Produto',
        help_text='Nome do produto (máx. 120 caracteres)'
    )
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        validators=[MinValueValidator(Decimal('0.01'))],
        verbose_name='Preço',
        help_text='Preço do produto em reais'
    )
    stock = models.IntegerField(
        default=0,
        validators=[MinValueValidator(0)],
        verbose_name='Estoque',
        help_text='Quantidade em estoque'
    )
    description = models.TextField(
        blank=True,
        null=True,
        verbose_name='Descrição',
        help_text='Descrição detalhada do produto'
    )
    is_active = models.BooleanField(
        default=True,
        verbose_name='Ativo',
        help_text='Produto ativo no sistema'
    )
    created_at = models.DateTimeField(
        auto_now_add=True,
        verbose_name='Criado em'
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        verbose_name='Atualizado em'
    )

    class Meta:
        ordering = ['-created_at']
        verbose_name = 'Produto'
        verbose_name_plural = 'Produtos'
        indexes = [
            models.Index(fields=['-created_at']),
            models.Index(fields=['name']),
        ]

    def __str__(self):
        return f"{self.name} - R$ {self.price}"
    
    @property
    def total_value(self):
        """Valor total em estoque"""
        return self.price * self.stock
    
    @property
    def stock_status(self):
        """Status do estoque"""
        if self.stock == 0:
            return 'out_of_stock'
        elif self.stock < 10:
            return 'low_stock'
        return 'in_stock'
```

```bash
# Criar e aplicar migrations
python manage.py makemigrations
python manage.py migrate
```

---

## 📝 4. Serializers

```python
# serializers.py
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    """Serializer padrão para produtos"""
    total_value = serializers.ReadOnlyField()
    stock_status = serializers.ReadOnlyField()
    
    class Meta:
        model = Product
        fields = [
            'id', 'name', 'price', 'stock', 'description',
            'is_active', 'created_at', 'updated_at',
            'total_value', 'stock_status'
        ]
        read_only_fields = ['id', 'created_at', 'updated_at']
    
    def validate_price(self, value):
        """Validação customizada de preço"""
        if value <= 0:
            raise serializers.ValidationError(
                "O preço deve ser maior que zero"
            )
        if value > 1000000:
            raise serializers.ValidationError(
                "Preço muito alto. Valor máximo: R$ 1.000.000,00"
            )
        return value
    
    def validate_stock(self, value):
        """Validação customizada de estoque"""
        if value < 0:
            raise serializers.ValidationError(
                "O estoque não pode ser negativo"
            )
        if value > 100000:
            raise serializers.ValidationError(
                "Estoque muito alto. Máximo: 100.000 unidades"
            )
        return value
    
    def validate(self, data):
        """Validação de múltiplos campos"""
        # Produtos caros não podem ter estoque muito alto
        price = data.get('price', 0)
        stock = data.get('stock', 0)
        
        if price > 10000 and stock > 100:
            raise serializers.ValidationError(
                "Produtos acima de R$ 10.000 não podem ter mais de 100 unidades em estoque"
            )
        
        return data


class ProductListSerializer(serializers.ModelSerializer):
    """Serializer simplificado para listagem"""
    stock_status = serializers.ReadOnlyField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'stock', 'stock_status']


class ProductDetailSerializer(serializers.ModelSerializer):
    """Serializer completo para detalhes"""
    total_value = serializers.ReadOnlyField()
    stock_status = serializers.ReadOnlyField()
    
    class Meta:
        model = Product
        fields = '__all__'
```

---

## 📄 5. Paginação Customizada

```python
# pagination.py
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response

class ProductPagination(PageNumberPagination):
    """Paginação customizada com mensagens claras"""
    page_size = 5
    page_size_query_param = 'page_size'
    max_page_size = 50
    page_query_param = 'page'

    def get_paginated_response(self, data):
        """Resposta paginada com informações extras"""
        return Response({
            "success": True,
            "message": "Lista de produtos carregada com sucesso",
            "pagination": {
                "total_items": self.page.paginator.count,
                "total_pages": self.page.paginator.num_pages,
                "current_page": self.page.number,
                "page_size": self.page_size,
                "has_next": self.page.has_next(),
                "has_previous": self.page.has_previous(),
                "next_page": self.page.next_page_number() if self.page.has_next() else None,
                "previous_page": self.page.previous_page_number() if self.page.has_previous() else None,
            },
            "results": data
        })
```

---

## 🔐 6. Permissões Customizadas

```python
# permissions.py
from rest_framework.permissions import BasePermission, SAFE_METHODS

class IsAdminOrReadOnly(BasePermission):
    """
    Permissão customizada:
    - Usuários comuns: apenas leitura (GET, HEAD, OPTIONS)
    - Admins (is_staff): acesso total
    """
    
    def has_permission(self, request, view):
        # Métodos seguros (leitura) são permitidos para todos autenticados
        if request.method in SAFE_METHODS:
            return request.user and request.user.is_authenticated
        
        # Métodos de escrita apenas para staff
        return request.user and request.user.is_staff
    
    def has_object_permission(self, request, view, obj):
        # Mesma lógica para permissões a nível de objeto
        if request.method in SAFE_METHODS:
            return True
        
        return request.user and request.user.is_staff


class IsAdminUser(BasePermission):
    """
    Permite acesso apenas para usuários admin
    """
    
    def has_permission(self, request, view):
        return request.user and request.user.is_staff
```

---

## 🎯 7. ViewSet Completo

```python
# views.py
from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.exceptions import ValidationError
from django_filters.rest_framework import DjangoFilterBackend
from django.db.models import Sum, Avg, Count

from .models import Product
from .serializers import (
    ProductSerializer,
    ProductListSerializer,
    ProductDetailSerializer
)
from .pagination import ProductPagination
from .permissions import IsAdminOrReadOnly


class ProductViewSet(viewsets.ModelViewSet):
    """
    ViewSet completo para gerenciamento de produtos
    
    Endpoints disponíveis:
    - GET /api/products/ - Lista produtos (todos)
    - POST /api/products/ - Cria produto (apenas admin)
    - GET /api/products/{id}/ - Detalhe do produto (todos)
    - PUT /api/products/{id}/ - Atualiza produto completamente (apenas admin)
    - PATCH /api/products/{id}/ - Atualiza produto parcialmente (apenas admin)
    - DELETE /api/products/{id}/ - Remove produto (apenas admin)
    - GET /api/products/low_stock/ - Produtos com estoque baixo (todos)
    - GET /api/products/statistics/ - Estatísticas gerais (todos)
    - POST /api/products/{id}/add_stock/ - Adiciona estoque (apenas admin)
    - POST /api/products/{id}/remove_stock/ - Remove estoque (apenas admin)
    """
    
    queryset = Product.objects.filter(is_active=True).order_by('-created_at')
    serializer_class = ProductSerializer
    
    # 🔐 Segurança
    permission_classes = [IsAuthenticated, IsAdminOrReadOnly]
    
    # 📄 Paginação
    pagination_class = ProductPagination
    
    # 🔍 Filtros / Pesquisa / Ordenação
    filter_backends = [
        DjangoFilterBackend,
        filters.SearchFilter,
        filters.OrderingFilter,
    ]
    
    # Filtros exatos
    filterset_fields = ['stock', 'is_active']
    
    # Campos para busca textual
    search_fields = ['name', 'description']
    
    # Campos permitidos para ordenação
    ordering_fields = ['price', 'stock', 'created_at', 'name']
    
    # ========== Serializers diferentes por ação ==========
    
    def get_serializer_class(self):
        """Retorna serializer apropriado para cada ação"""
        if self.action == 'list':
            return ProductListSerializer
        elif self.action == 'retrieve':
            return ProductDetailSerializer
        return ProductSerializer
    
    # ========== QuerySet customizado ==========
    
    def get_queryset(self):
        """QuerySet customizado baseado em parâmetros"""
        queryset = super().get_queryset()
        
        # Filtro por estoque disponível
        in_stock = self.request.query_params.get('in_stock', None)
        if in_stock and in_stock.lower() == 'true':
            queryset = queryset.filter(stock__gt=0)
        
        # Filtro por faixa de preço
        min_price = self.request.query_params.get('min_price', None)
        max_price = self.request.query_params.get('max_price', None)
        
        if min_price:
            queryset = queryset.filter(price__gte=min_price)
        if max_price:
            queryset = queryset.filter(price__lte=max_price)
        
        return queryset
    
    # ========== Mensagens customizadas em métodos padrão ==========
    
    def list(self, request, *args, **kwargs):
        """GET /api/products/ - Lista produtos com paginação"""
        # A paginação já adiciona a mensagem
        return super().list(request, *args, **kwargs)
    
    def retrieve(self, request, *args, **kwargs):
        """GET /api/products/{id}/ - Detalhe de um produto"""
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        
        return Response({
            "success": True,
            "message": "Produto encontrado com sucesso",
            "data": serializer.data
        })
    
    def create(self, request, *args, **kwargs):
        """POST /api/products/ - Cria novo produto"""
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        
        return Response({
            "success": True,
            "message": "Produto criado com sucesso",
            "data": serializer.data
        }, status=status.HTTP_201_CREATED)
    
    def update(self, request, *args, **kwargs):
        """PUT /api/products/{id}/ - Atualização completa"""
        partial = kwargs.pop('partial', False)
        instance = self.get_object()
        serializer = self.get_serializer(instance, data=request.data, partial=partial)
        serializer.is_valid(raise_exception=True)
        self.perform_update(serializer)
        
        return Response({
            "success": True,
            "message": "Produto atualizado com sucesso",
            "data": serializer.data
        })
    
    def partial_update(self, request, *args, **kwargs):
        """PATCH /api/products/{id}/ - Atualização parcial"""
        kwargs['partial'] = True
        return self.update(request, *args, **kwargs)
    
    def destroy(self, request, *args, **kwargs):
        """DELETE /api/products/{id}/ - Remove produto"""
        instance = self.get_object()
        product_name = instance.name
        
        # Soft delete (apenas marca como inativo)
        instance.is_active = False
        instance.save()
        
        # Ou delete permanente:
        # self.perform_destroy(instance)
        
        return Response({
            "success": True,
            "message": f"Produto '{product_name}' removido com sucesso"
        }, status=status.HTTP_200_OK)
    
    # ========== Validações antes de salvar ==========
    
    def perform_create(self, serializer):
        """Hook executado antes de criar"""
        # Validações adicionais se necessário
        serializer.save()
    
    def perform_update(self, serializer):
        """Hook executado antes de atualizar"""
        # Validações adicionais se necessário
        serializer.save()
    
    def perform_destroy(self, instance):
        """Hook executado antes de deletar"""
        instance.delete()
    
    # ========== Ações customizadas ==========
    
    @action(detail=False, methods=['get'], permission_classes=[IsAuthenticated])
    def low_stock(self, request):
        """
        GET /api/products/low_stock/
        Retorna produtos com estoque baixo (< 10 unidades)
        """
        products = self.get_queryset().filter(stock__lt=10)
        serializer = self.get_serializer(products, many=True)
        
        return Response({
            "success": True,
            "message": f"{products.count()} produtos com estoque baixo",
            "count": products.count(),
            "results": serializer.data
        })
    
    @action(detail=False, methods=['get'], permission_classes=[IsAuthenticated])
    def statistics(self, request):
        """
        GET /api/products/statistics/
        Retorna estatísticas gerais dos produtos
        """
        queryset = self.get_queryset()
        
        stats = queryset.aggregate(
            total_products=Count('id'),
            total_stock=Sum('stock'),
            average_price=Avg('price'),
        )
        
        return Response({
            "success": True,
            "message": "Estatísticas carregadas com sucesso",
            "data": {
                "total_products": stats['total_products'] or 0,
                "total_stock": stats['total_stock'] or 0,
                "average_price": round(float(stats['average_price'] or 0), 2),
                "low_stock_items": queryset.filter(stock__lt=10).count(),
                "out_of_stock_items": queryset.filter(stock=0).count(),
            }
        })
    
    @action(detail=True, methods=['post'], permission_classes=[IsAuthenticated, IsAdminOrReadOnly])
    def add_stock(self, request, pk=None):
        """
        POST /api/products/{id}/add_stock/
        Adiciona estoque a um produto
        
        Body: {"amount": 10}
        """
        product = self.get_object()
        amount = request.data.get('amount', 0)
        
        # Validações
        if not isinstance(amount, (int, float)):
            return Response({
                "success": False,
                "message": "A quantidade deve ser um número"
            }, status=status.HTTP_400_BAD_REQUEST)
        
        if amount <= 0:
            return Response({
                "success": False,
                "message": "A quantidade deve ser maior que zero"
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Atualiza estoque
        product.stock += int(amount)
        product.save()
        
        return Response({
            "success": True,
            "message": f"{amount} unidades adicionadas ao estoque",
            "data": {
                "product_id": product.id,
                "product_name": product.name,
                "amount_added": amount,
                "current_stock": product.stock
            }
        })
    
    @action(detail=True, methods=['post'], permission_classes=[IsAuthenticated, IsAdminOrReadOnly])
    def remove_stock(self, request, pk=None):
        """
        POST /api/products/{id}/remove_stock/
        Remove estoque de um produto
        
        Body: {"amount": 5}
        """
        product = self.get_object()
        amount = request.data.get('amount', 0)
        
        # Validações
        if not isinstance(amount, (int, float)):
            return Response({
                "success": False,
                "message": "A quantidade deve ser um número"
            }, status=status.HTTP_400_BAD_REQUEST)
        
        if amount <= 0:
            return Response({
                "success": False,
                "message": "A quantidade deve ser maior que zero"
            }, status=status.HTTP_400_BAD_REQUEST)
        
        if product.stock < amount:
            return Response({
                "success": False,
                "message": f"Estoque insuficiente. Disponível: {product.stock} unidades"
            }, status=status.HTTP_400_BAD_REQUEST)
        
        # Atualiza estoque
        product.stock -= int(amount)
        product.save()
        
        return Response({
            "success": True,
            "message": f"{amount} unidades removidas do estoque",
            "data": {
                "product_id": product.id,
                "product_name": product.name,
                "amount_removed": amount,
                "current_stock": product.stock
            }
        })
```

---

## 🛣️ 8. URLs

```python
# products/urls.py
from rest_framework.routers import DefaultRouter
from django.urls import path, include
from .views import ProductViewSet

router = DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')

urlpatterns = [
    path('', include(router.urls)),
]
```

```python
# project/urls.py (URL principal do projeto)
from django.contrib import admin
from django.urls import path, include
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)

urlpatterns = [
    path('admin/', admin.site.urls),
    
    # JWT Authentication
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
    
    # API endpoints
    path('api/', include('products.urls')),
]
```

---

## 📊 9. Tabela de Permissões

| Usuário              | GET (lista/detalhe) | POST (criar) | PUT/PATCH (atualizar) | DELETE (remover) |
|----------------------|---------------------|--------------|----------------------|------------------|
| **Não autenticado**  | ❌                   | ❌            | ❌                    | ❌                |
| **Usuário comum**    | ✅                   | ❌            | ❌                    | ❌                |
| **Admin (is_staff)** | ✅                   | ✅            | ✅                    | ✅                |

**Ações customizadas:**
- `low_stock` - ✅ Todos autenticados
- `statistics` - ✅ Todos autenticados  
- `add_stock` - ✅ Apenas admins
- `remove_stock` - ✅ Apenas admins

---

## 🚀 10. Exemplos de Uso (com curl)

### **1. Obter Token JWT (Login)**

```bash
curl -X POST http://localhost:8000/api/token/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "senha123"
  }'
```

**Resposta:**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

### **2. Listar Produtos (com paginação)**

```bash
curl -X GET "http://localhost:8000/api/products/?page=1&page_size=5" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."
```

**Resposta:**
```json
{
  "success": true,
  "message": "Lista de produtos carregada com sucesso",
  "pagination": {
    "total_items": 25,
    "total_pages": 5,
    "current_page": 1,
    "page_size": 5,
    "has_next": true,
    "has_previous": false,
    "next_page": 2,
    "previous_page": null
  },
  "results": [...]
}
```

### **3. Buscar Produtos por Nome**

```bash
curl -X GET "http://localhost:8000/api/products/?search=laptop" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."
```

### **4. Filtrar por Faixa de Preço**

```bash
curl -X GET "http://localhost:8000/api/products/?min_price=100&max_price=500" \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."
```

### **5. Criar Produto (apenas admin)**

```bash
curl -X POST http://localhost:8000/api/products/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Laptop Dell",
    "price": 2500.00,
    "stock": 10,
    "description": "Laptop Dell Inspiron 15"
  }'
```

**Resposta:**
```json
{
  "success": true,
  "message": "Produto criado com sucesso",
  "data": {
    "id": 1,
    "name": "Laptop Dell",
    "price": "2500.00",
    "stock": 10,
    ...
  }
}
```

### **6. Atualizar Produto Parcialmente (PATCH)**

```bash
curl -X PATCH http://localhost:8000/api/products/1/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "price": 2300.00
  }'
```

### **7. Adicionar Estoque**

```bash
curl -X POST http://localhost:8000/api/products/1/add_stock/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..." \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 20
  }'
```

### **8. Ver Estatísticas**

```bash
curl -X GET http://localhost:8000/api/products/statistics/ \
  -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc..."
```

### **9. Refresh Token**

```bash
curl -X POST http://localhost:8000/api/token/refresh/ \
  -H "Content-Type: application/json" \
  -d '{
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }'
```

---

## 🧪 11. Testes Automatizados

```python
# tests.py
from rest_framework.test import APITestCase, APIClient
from rest_framework import status
from django.contrib.auth.models import User
from .models import Product
from decimal import Decimal

class ProductAPITestCase(APITestCase):
    """Testes para a API de produtos"""
    
    def setUp(self):
        """Configuração inicial dos testes"""
        # Criar usuário admin
        self.admin = User.objects.create_superuser(
            username='admin',
            email='admin@test.com',
            password='admin123'
        )
        
        # Criar usuário comum
        self.user = User.objects.create_user(
            username='user',
            email='user@test.com',
            password='user123'
        )
        
        # Criar produtos de teste
        self.product1 = Product.objects.create(
            name='Laptop Dell',
            price=Decimal('2500.00'),
            stock=10
        )
        
        self.product2 = Product.objects.create(
            name='Mouse Logitech',
            price=Decimal('50.00'),
            stock=5
        )
        
        # Cliente API
        self.client = APIClient()
    
    # ========== Testes de Autenticação ==========
    
    def test_unauthenticated_user_cannot_access(self):
        """Usuário não autenticado não pode acessar"""
        response = self.client.get('/api/products/')
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    # ========== Testes de Listagem ==========
    
    def test_user_can_list_products(self):
        """Usuário comum pode listar produtos"""
        self.client.force_authenticate(user=self.user)
        response = self.client.get('/api/products/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
        self.assertIn('results', response.data)
    
    def test_admin_can_list_products(self):
        """Admin pode listar produtos"""
        self.client.force_authenticate(user=self.admin)
        response = self.client.get('/api/products/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
    
    # ========== Testes de Detalhes ==========
    
    def test_user_can_retrieve_product(self):
        """Usuário comum pode ver detalhes"""
        self.client.force_authenticate(user=self.user)
        response = self.client.get(f'/api/products/{self.product1.id}/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
        self.assertEqual(response.data['data']['name'], 'Laptop Dell')
    
    # ========== Testes de Criação ==========
    
    def test_user_cannot_create_product(self):
        """Usuário comum NÃO pode criar produtos"""
        self.client.force_authenticate(user=self.user)
        data = {
            'name': 'Teclado Mecânico',
            'price': 300.00,
            'stock': 15
        }
        response = self.client.post('/api/products/', data)
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_admin_can_create_product(self):
        """Admin pode criar produtos"""
        self.client.force_authenticate(user=self.admin)
        data = {
            'name': 'Teclado Mecânico',
            'price': 300.00,
            'stock': 15
        }
        response = self.client.post('/api/products/', data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertTrue(response.data['success'])
        self.assertEqual(response.data['data']['name'], 'Teclado Mecânico')
    
    def test_cannot_create_product_with_negative_price(self):
        """Não pode criar produto com preço negativo"""
        self.client.force_authenticate(user=self.admin)
        data = {
            'name': 'Produto Inválido',
            'price': -100.00,
            'stock': 10
        }
        response = self.client.post('/api/products/', data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    # ========== Testes de Atualização ==========
    
    def test_user_cannot_update_product(self):
        """Usuário comum NÃO pode atualizar"""
        self.client.force_authenticate(user=self.user)
        data = {'price': 2000.00}
        response = self.client.patch(
            f'/api/products/{self.product1.id}/', 
            data, 
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_admin_can_update_product(self):
        """Admin pode atualizar produtos"""
        self.client.force_authenticate(user=self.admin)
        data = {'price': 2000.00}
        response = self.client.patch(
            f'/api/products/{self.product1.id}/', 
            data, 
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
    
    # ========== Testes de Remoção ==========
    
    def test_user_cannot_delete_product(self):
        """Usuário comum NÃO pode deletar"""
        self.client.force_authenticate(user=self.user)
        response = self.client.delete(f'/api/products/{self.product1.id}/')
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_admin_can_delete_product(self):
        """Admin pode deletar produtos"""
        self.client.force_authenticate(user=self.admin)
        response = self.client.delete(f'/api/products/{self.product2.id}/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
    
    # ========== Testes de Ações Customizadas ==========
    
    def test_low_stock_action(self):
        """Testa ação de produtos com estoque baixo"""
        self.client.force_authenticate(user=self.user)
        response = self.client.get('/api/products/low_stock/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
        self.assertIn('count', response.data)
    
    def test_statistics_action(self):
        """Testa ação de estatísticas"""
        self.client.force_authenticate(user=self.user)
        response = self.client.get('/api/products/statistics/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
        self.assertIn('data', response.data)
        self.assertIn('total_products', response.data['data'])
    
    def test_user_cannot_add_stock(self):
        """Usuário comum não pode adicionar estoque"""
        self.client.force_authenticate(user=self.user)
        data = {'amount': 10}
        response = self.client.post(
            f'/api/products/{self.product1.id}/add_stock/',
            data,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_admin_can_add_stock(self):
        """Admin pode adicionar estoque"""
        self.client.force_authenticate(user=self.admin)
        data = {'amount': 10}
        response = self.client.post(
            f'/api/products/{self.product1.id}/add_stock/',
            data,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
        
        # Verifica se o estoque foi atualizado
        self.product1.refresh_from_db()
        self.assertEqual(self.product1.stock, 20)
    
    # ========== Testes de Filtros ==========
    
    def test_search_filter(self):
        """Testa busca por nome"""
        self.client.force_authenticate(user=self.user)
        response = self.client.get('/api/products/?search=laptop')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        # Deve retornar apenas o laptop
        self.assertEqual(len(response.data['results']), 1)
    
    def test_price_range_filter(self):
        """Testa filtro por faixa de preço"""
        self.client.force_authenticate(user=self.user)
        response = self.client.get('/api/products/?min_price=40&max_price=60')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        # Deve retornar apenas o mouse
        self.assertEqual(len(response.data['results']), 1)
```

**Executar testes:**
```bash
python manage.py test products
```

---

## 👨‍💼 12. Criar Usuários de Teste

```bash
# Via shell do Django
python manage.py shell
```

```python
from django.contrib.auth.models import User

# Criar admin
admin = User.objects.create_superuser(
    username='admin',
    email='admin@example.com',
    password='admin123'
)

# Criar usuário comum
user = User.objects.create_user(
    username='user',
    email='user@example.com',
    password='user123'
)
```

---

## 📱 13. Exemplo Frontend (JavaScript)

```javascript
// config.js
const API_URL = 'http://localhost:8000/api';
let accessToken = null;
let refreshToken = null;

// Login
async function login(username, password) {
    const response = await fetch(`${API_URL}/token/`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({ username, password })
    });
    
    const data = await response.json();
    accessToken = data.access;
    refreshToken = data.refresh;
    
    // Salvar no localStorage (apenas para exemplo)
    localStorage.setItem('access_token', accessToken);
    localStorage.setItem('refresh_token', refreshToken);
    
    return data;
}

// Refresh token
async function refreshAccessToken() {
    const response = await fetch(`${API_URL}/token/refresh/`, {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({ refresh: refreshToken })
    });
    
    const data = await response.json();
    accessToken = data.access;
    localStorage.setItem('access_token', accessToken);
    
    return data;
}

// Fazer requisição autenticada
async function apiRequest(endpoint, options = {}) {
    const headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
        ...options.headers
    };
    
    const response = await fetch(`${API_URL}${endpoint}`, {
        ...options,
        headers
    });
    
    // Se token expirou, tenta refresh
    if (response.status === 401) {
        await refreshAccessToken();
        // Tenta novamente com novo token
        return apiRequest(endpoint, options);
    }
    
    return response.json();
}

// Listar produtos
async function getProducts(page = 1) {
    return await apiRequest(`/products/?page=${page}`);
}

// Criar produto (apenas admin)
async function createProduct(productData) {
    return await apiRequest('/products/', {
        method: 'POST',
        body: JSON.stringify(productData)
    });
}

// Atualizar produto (apenas admin)
async function updateProduct(id, productData) {
    return await apiRequest(`/products/${id}/`, {
        method: 'PATCH',
        body: JSON.stringify(productData)
    });
}

// Deletar produto (apenas admin)
async function deleteProduct(id) {
    return await apiRequest(`/products/${id}/`, {
        method: 'DELETE'
    });
}

// Exemplo de uso:
// login('admin', 'admin123').then(() => {
//     getProducts().then(data => console.log(data));
// });
```

---

## 🎯 14. Verdades Sem Filtro

### ✅ **O que realmente importa:**

1. **90% dos projetos precisam apenas de:**
   - `ModelViewSet` bem configurado
   - Permissões corretamente implementadas
   - Validações no serializer
   - Mensagens claras nas respostas

2. **Use `ViewSet` puro apenas quando:**
   - A lógica é muito específica
   - Não existe modelo direto
   - Precisa agrupar ações não-CRUD

3. **JWT sem HttpOnly refresh é meia segurança:**
   - Access token no localStorage pode ser roubado via XSS
   - Refresh token deve estar em cookie HttpOnly
   - Considere usar bibliotecas como `dj-rest-auth` para isso

4. **Permissão mal feita = brecha de produção:**
   - SEMPRE teste permissões
   - Use `has_permission` E `has_object_permission`
   - Não confie apenas em `is_authenticated`

5. **Respostas customizadas facilitam frontend:**
   - Sempre retorne `{"success": true/false, "message": "...", "data": {}}`
   - Frontend não precisa adivinhar o que aconteceu
   - Mensagens em português para usuários brasileiros

6. **Paginação é obrigatória em produção:**
   - Nunca retorne todos os registros sem paginação
   - Use `PageNumberPagination` ou `CursorPagination`
   - Informe total de páginas e itens

7. **Validação em cascata:**
   - Validação no modelo (integridade de dados)
   - Validação no serializer (lógica de negócio)
   - Validação na view (contexto da requisição)

---

## 🚀 15. Próximos Passos

1. **Documentação automática com Swagger:**
   ```bash
   pip install drf-spectacular
   ```

2. **Throttling (limitação de taxa):**
   ```python
   REST_FRAMEWORK = {
       'DEFAULT_THROTTLE_CLASSES': [
           'rest_framework.throttling.AnonRateThrottle',
           'rest_framework.throttling.UserRateThrottle'
       ],
       'DEFAULT_THROTTLE_RATES': {
           'anon': '100/day',
           'user': '1000/day'
       }
   }
   ```

3. **CORS para frontend separado:**
   ```bash
   pip install django-cors-headers
   ```

4. **Logging e monitoramento:**
   - Use `django-silk` para debug
   - Configure logs do Django
   - Monitore com Sentry em produção

5. **Cache:**
   - Redis para cache de queries
   - `@method_decorator(cache_page(60*15))` para views

---

**Versão:** 3.0 - Production Ready 🚀
**Status:** Completo e testado ✅