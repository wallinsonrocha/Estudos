# Guia Prático: ViewSets no Django REST Framework

## 📦 Setup Inicial

Antes de começar, vamos definir nosso modelo base que será usado em todos os exemplos:

```python
# models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.IntegerField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    def __str__(self):
        return self.name
    
    class Meta:
        ordering = ['-created_at']
```

---

# 1️⃣ CRUD Completo com `ModelViewSet`

O `ModelViewSet` é a forma mais rápida de criar uma API CRUD completa.

## Serializer Básico

```python
# serializers.py
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'stock', 'created_at', 'updated_at']
        read_only_fields = ['id', 'created_at', 'updated_at']
```

## ViewSet Padrão (CRUD Completo)

```python
# views.py
from rest_framework import viewsets
from rest_framework import status
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    """
    ViewSet para operações CRUD de produtos.
    
    Fornece automaticamente:
    - list: GET /products/
    - create: POST /products/
    - retrieve: GET /products/{id}/
    - update: PUT /products/{id}/
    - partial_update: PATCH /products/{id}/
    - destroy: DELETE /products/{id}/
    """
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

## Router para URLs Automáticas

```python
# urls.py
from rest_framework.routers import DefaultRouter
from django.urls import path, include
from .views import ProductViewSet

router = DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')

urlpatterns = [
    path('api/', include(router.urls)),
]
```

**Resultado:** URLs criadas automaticamente:

* `GET /api/products/` → lista todos os produtos
* `POST /api/products/` → cria novo produto
* `GET /api/products/{id}/` → detalhe de um produto
* `PUT /api/products/{id}/` → atualização completa
* `PATCH /api/products/{id}/` → atualização parcial
* `DELETE /api/products/{id}/` → deleta produto

---

# 2️⃣ Customizando Métodos do `ModelViewSet`

Você pode sobrescrever os métodos padrão para adicionar lógica customizada.

```python
from rest_framework import viewsets, status
from rest_framework.response import Response
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def create(self, request, *args, **kwargs):
        """
        POST /products/
        Customiza a criação de produtos com validações extras
        """
        # Validação customizada antes de salvar
        if request.data.get('price', 0) <= 0:
            return Response(
                {"error": "Preço deve ser maior que zero"}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        
        if request.data.get('stock', 0) < 0:
            return Response(
                {"error": "Estoque não pode ser negativo"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        # Chama o método original
        response = super().create(request, *args, **kwargs)
        
        # Adiciona mensagem customizada na resposta
        response.data['message'] = "Produto criado com sucesso!"
        
        return response

    def update(self, request, *args, **kwargs):
        """
        PUT /products/{id}/
        Atualização completa com validações
        """
        # Validação antes de atualizar
        if request.data.get('stock', 0) < 0:
            return Response(
                {"error": "Estoque não pode ser negativo"}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        
        return super().update(request, *args, **kwargs)
    
    def partial_update(self, request, *args, **kwargs):
        """
        PATCH /products/{id}/
        Atualização parcial
        """
        # Validações específicas para PATCH
        if 'price' in request.data and request.data['price'] <= 0:
            return Response(
                {"error": "Preço deve ser maior que zero"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        return super().partial_update(request, *args, **kwargs)

    def destroy(self, request, *args, **kwargs):
        """
        DELETE /products/{id}/
        Impede exclusão de produtos com estoque
        """
        instance = self.get_object()
        
        if instance.stock > 0:
            return Response(
                {"error": "Não pode deletar produtos em estoque"}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        
        return super().destroy(request, *args, **kwargs)
    
    def list(self, request, *args, **kwargs):
        """
        GET /products/
        Customiza a listagem
        """
        response = super().list(request, *args, **kwargs)
        
        # Adiciona metadados à resposta
        response.data['total_products'] = Product.objects.count()
        
        return response
    
    def retrieve(self, request, *args, **kwargs):
        """
        GET /products/{id}/
        Customiza o retorno de detalhes
        """
        instance = self.get_object()
        serializer = self.get_serializer(instance)
        
        # Adiciona informações extras
        data = serializer.data
        data['total_value'] = instance.price * instance.stock
        
        return Response(data)
```

---

# 3️⃣ `ViewSet` Puro - Controle Total

Quando você precisa de controle absoluto sobre cada ação, use `ViewSet` puro.

```python
from rest_framework import viewsets, status
from rest_framework.response import Response
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ViewSet):
    """
    ViewSet puro onde você implementa cada ação manualmente.
    Útil quando a lógica é muito específica e não segue padrões CRUD.
    """

    def list(self, request):
        """GET /products/ - Lista produtos"""
        products = Product.objects.all()
        serializer = ProductSerializer(products, many=True)
        return Response({
            'count': products.count(),
            'results': serializer.data
        })

    def retrieve(self, request, pk=None):
        """GET /products/{id}/ - Detalhe de um produto"""
        try:
            product = Product.objects.get(pk=pk)
        except Product.DoesNotExist:
            return Response(
                {"error": "Produto não encontrado"}, 
                status=status.HTTP_404_NOT_FOUND
            )
        
        serializer = ProductSerializer(product)
        return Response(serializer.data)

    def create(self, request):
        """POST /products/ - Cria produto"""
        serializer = ProductSerializer(data=request.data)
        
        if serializer.is_valid():
            serializer.save()
            return Response(
                serializer.data, 
                status=status.HTTP_201_CREATED
            )
        
        return Response(
            serializer.errors, 
            status=status.HTTP_400_BAD_REQUEST
        )

    def update(self, request, pk=None):
        """PUT /products/{id}/ - Atualização completa"""
        try:
            product = Product.objects.get(pk=pk)
        except Product.DoesNotExist:
            return Response(
                {"error": "Produto não encontrado"}, 
                status=status.HTTP_404_NOT_FOUND
            )
        
        serializer = ProductSerializer(product, data=request.data)
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        
        return Response(
            serializer.errors, 
            status=status.HTTP_400_BAD_REQUEST
        )
    
    def partial_update(self, request, pk=None):
        """PATCH /products/{id}/ - Atualização parcial"""
        try:
            product = Product.objects.get(pk=pk)
        except Product.DoesNotExist:
            return Response(
                {"error": "Produto não encontrado"}, 
                status=status.HTTP_404_NOT_FOUND
            )
        
        # partial=True permite atualização parcial
        serializer = ProductSerializer(product, data=request.data, partial=True)
        
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        
        return Response(
            serializer.errors, 
            status=status.HTTP_400_BAD_REQUEST
        )

    def destroy(self, request, pk=None):
        """DELETE /products/{id}/ - Deleta produto"""
        try:
            product = Product.objects.get(pk=pk)
        except Product.DoesNotExist:
            return Response(
                {"error": "Produto não encontrado"}, 
                status=status.HTTP_404_NOT_FOUND
            )
        
        product.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

**✅ Diferença Principal:**

* **`ViewSet`** → você implementa tudo manualmente, total controle.
* **`ModelViewSet`** → DRF faz CRUD automático, você só sobrescreve se necessário.

---

# 4️⃣ Ações Customizadas com `@action`

Uma das funcionalidades mais poderosas dos ViewSets.

```python
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django.db.models import Sum, Avg
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    @action(detail=False, methods=['get'])
    def low_stock(self, request):
        """
        GET /products/low_stock/
        Retorna produtos com estoque baixo (< 10 unidades)
        """
        products = Product.objects.filter(stock__lt=10)
        serializer = self.get_serializer(products, many=True)
        return Response({
            'count': products.count(),
            'products': serializer.data
        })
    
    @action(detail=False, methods=['get'])
    def expensive(self, request):
        """
        GET /products/expensive/
        Retorna produtos acima de R$ 1000
        """
        products = Product.objects.filter(price__gt=1000)
        serializer = self.get_serializer(products, many=True)
        return Response(serializer.data)
    
    @action(detail=True, methods=['post'])
    def add_stock(self, request, pk=None):
        """
        POST /products/{id}/add_stock/
        Adiciona estoque a um produto específico
        
        Body: {"amount": 10}
        """
        product = self.get_object()
        amount = request.data.get('amount', 0)
        
        if not isinstance(amount, (int, float)) or amount <= 0:
            return Response(
                {"error": "Quantidade deve ser um número positivo"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        product.stock += int(amount)
        product.save()
        
        return Response({
            'message': 'Estoque atualizado com sucesso',
            'product': self.get_serializer(product).data,
            'amount_added': amount,
            'current_stock': product.stock
        })
    
    @action(detail=True, methods=['post'])
    def remove_stock(self, request, pk=None):
        """
        POST /products/{id}/remove_stock/
        Remove estoque de um produto específico
        
        Body: {"amount": 5}
        """
        product = self.get_object()
        amount = request.data.get('amount', 0)
        
        if not isinstance(amount, (int, float)) or amount <= 0:
            return Response(
                {"error": "Quantidade deve ser um número positivo"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        if product.stock < amount:
            return Response(
                {"error": f"Estoque insuficiente. Disponível: {product.stock}"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        product.stock -= int(amount)
        product.save()
        
        return Response({
            'message': 'Estoque removido com sucesso',
            'amount_removed': amount,
            'current_stock': product.stock
        })
    
    @action(detail=False, methods=['get'])
    def statistics(self, request):
        """
        GET /products/statistics/
        Retorna estatísticas gerais dos produtos
        """
        stats = Product.objects.aggregate(
            total_products=Sum('stock'),
            average_price=Avg('price'),
            total_value=Sum('price') * Sum('stock')
        )
        
        return Response({
            'total_products': Product.objects.count(),
            'total_stock': stats['total_products'],
            'average_price': stats['average_price'],
            'low_stock_items': Product.objects.filter(stock__lt=10).count(),
        })
    
    @action(detail=True, methods=['post'])
    def apply_discount(self, request, pk=None):
        """
        POST /products/{id}/apply_discount/
        Aplica desconto percentual ao produto
        
        Body: {"discount_percent": 10}
        """
        product = self.get_object()
        discount = request.data.get('discount_percent', 0)
        
        if not isinstance(discount, (int, float)) or discount <= 0 or discount > 100:
            return Response(
                {"error": "Desconto deve ser entre 0 e 100"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        original_price = product.price
        discount_amount = original_price * (discount / 100)
        product.price = original_price - discount_amount
        product.save()
        
        return Response({
            'message': 'Desconto aplicado com sucesso',
            'original_price': str(original_price),
            'discount_percent': discount,
            'discount_amount': str(discount_amount),
            'new_price': str(product.price)
        })
```

**Parâmetros do `@action`:**

* `detail=False` → ação em nível de coleção (`/products/action/`)
* `detail=True` → ação em nível de objeto (`/products/{id}/action/`)
* `methods=['get', 'post']` → métodos HTTP permitidos
* `url_path='custom-url'` → customizar a URL (opcional)
* `url_name='custom_name'` → nome para reverse (opcional)

---

# 5️⃣ Filtros, Pesquisa e Paginação

```python
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters
from rest_framework.pagination import PageNumberPagination

class ProductPagination(PageNumberPagination):
    """Configuração de paginação customizada"""
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    pagination_class = ProductPagination
    
    # Backends de filtro
    filter_backends = [
        DjangoFilterBackend,    # Filtros exatos
        filters.SearchFilter,    # Busca por texto
        filters.OrderingFilter   # Ordenação
    ]
    
    # Filtros exatos por campo
    filterset_fields = ['stock', 'price']
    
    # Campos para busca por texto (usa LIKE no SQL)
    search_fields = ['name']
    
    # Campos permitidos para ordenação
    ordering_fields = ['price', 'stock', 'created_at', 'name']
    
    # Ordenação padrão
    ordering = ['-created_at']
```

**Exemplos de uso:**

```bash
# Paginação
GET /products/?page=2
GET /products/?page=1&page_size=20

# Filtros exatos
GET /products/?stock=10
GET /products/?price=99.90
GET /products/?stock=5&price=150.00

# Busca por texto
GET /products/?search=laptop
GET /products/?search=samsung

# Ordenação
GET /products/?ordering=price          # Preço crescente
GET /products/?ordering=-price         # Preço decrescente
GET /products/?ordering=-stock,price   # Por estoque desc, depois preço asc

# Combinando tudo
GET /products/?search=phone&ordering=-price&page=1&page_size=5
```

**Filtros mais avançados (requer django-filter):**

```python
from django_filters import rest_framework as filters

class ProductFilter(filters.FilterSet):
    """Filtros customizados mais complexos"""
    min_price = filters.NumberFilter(field_name="price", lookup_expr='gte')
    max_price = filters.NumberFilter(field_name="price", lookup_expr='lte')
    min_stock = filters.NumberFilter(field_name="stock", lookup_expr='gte')
    name_contains = filters.CharFilter(field_name="name", lookup_expr='icontains')
    
    class Meta:
        model = Product
        fields = ['min_price', 'max_price', 'min_stock', 'name_contains']

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filterset_class = ProductFilter  # Usa a classe customizada
```

**Uso dos filtros avançados:**

```bash
GET /products/?min_price=100&max_price=500
GET /products/?min_stock=10
GET /products/?name_contains=laptop
```

---

# 6️⃣ Autenticação e Permissões

```python
from rest_framework.permissions import (
    IsAuthenticated, 
    IsAdminUser, 
    AllowAny,
    IsAuthenticatedOrReadOnly
)
from rest_framework import viewsets

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def get_permissions(self):
        """
        Define permissões diferentes por ação
        """
        if self.action in ['list', 'retrieve']:
            # Qualquer pessoa pode ver produtos
            permission_classes = [AllowAny]
        
        elif self.action in ['create', 'update', 'partial_update', 'destroy']:
            # Apenas administradores podem modificar
            permission_classes = [IsAdminUser]
        
        else:
            # Ações customizadas requerem autenticação
            permission_classes = [IsAuthenticated]
        
        return [permission() for permission in permission_classes]
```

**Permissões simples (todas as ações):**

```python
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]  # Todas as ações requerem login
```

**Permissão customizada:**

```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Permite edição apenas ao dono do objeto
    """
    def has_object_permission(self, request, view, obj):
        # Leitura permitida para qualquer requisição
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Escrita apenas se o usuário for o dono
        return obj.owner == request.user

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsOwnerOrReadOnly]
```

---

# 7️⃣ Serializers Diferentes por Ação

Use serializers diferentes para listagem (mais leve) vs detalhes (mais completo).

```python
# serializers.py
class ProductListSerializer(serializers.ModelSerializer):
    """Serializer simplificado para listagem"""
    class Meta:
        model = Product
        fields = ['id', 'name', 'price']

class ProductDetailSerializer(serializers.ModelSerializer):
    """Serializer completo para detalhes"""
    total_value = serializers.SerializerMethodField()
    stock_status = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = [
            'id', 'name', 'price', 'stock', 
            'created_at', 'updated_at', 
            'total_value', 'stock_status'
        ]
    
    def get_total_value(self, obj):
        return obj.price * obj.stock
    
    def get_stock_status(self, obj):
        if obj.stock == 0:
            return 'out_of_stock'
        elif obj.stock < 10:
            return 'low_stock'
        return 'in_stock'

class ProductCreateUpdateSerializer(serializers.ModelSerializer):
    """Serializer para criação/atualização com validações"""
    class Meta:
        model = Product
        fields = ['name', 'price', 'stock']
    
    def validate_price(self, value):
        if value <= 0:
            raise serializers.ValidationError("Preço deve ser maior que zero")
        return value
    
    def validate_stock(self, value):
        if value < 0:
            raise serializers.ValidationError("Estoque não pode ser negativo")
        return value

# views.py
class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    
    def get_serializer_class(self):
        """Retorna serializer apropriado para cada ação"""
        if self.action == 'list':
            return ProductListSerializer
        elif self.action in ['create', 'update', 'partial_update']:
            return ProductCreateUpdateSerializer
        return ProductDetailSerializer
```

---

# 8️⃣ QuerySet Dinâmico

Modifique o queryset baseado em parâmetros da requisição ou usuário.

```python
class ProductViewSet(viewsets.ModelViewSet):
    serializer_class = ProductSerializer
    
    def get_queryset(self):
        """
        Retorna queryset customizado baseado no usuário ou parâmetros
        """
        queryset = Product.objects.all()
        
        # Filtrar por usuário (se o produto tiver campo owner)
        if not self.request.user.is_staff:
            queryset = queryset.filter(owner=self.request.user)
        
        # Filtrar por parâmetros de URL
        category = self.request.query_params.get('category', None)
        if category:
            queryset = queryset.filter(category=category)
        
        # Apenas produtos em estoque
        in_stock = self.request.query_params.get('in_stock', None)
        if in_stock:
            queryset = queryset.filter(stock__gt=0)
        
        return queryset
```

---

# 9️⃣ Tratamento de Erros Customizado

```python
from rest_framework.exceptions import ValidationError
from django.core.exceptions import ObjectDoesNotExist

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    def perform_create(self, serializer):
        """
        Hook executado antes de salvar na criação
        """
        # Validação customizada
        if serializer.validated_data['stock'] == 0:
            raise ValidationError("Não pode criar produto sem estoque")
        
        serializer.save()
    
    def perform_update(self, serializer):
        """
        Hook executado antes de salvar na atualização
        """
        instance = serializer.instance
        
        # Exemplo: não permitir reduzir preço em mais de 50%
        new_price = serializer.validated_data.get('price', instance.price)
        if new_price < instance.price * 0.5:
            raise ValidationError("Não pode reduzir preço em mais de 50%")
        
        serializer.save()
    
    def perform_destroy(self, instance):
        """
        Hook executado antes de deletar
        """
        # Soft delete ao invés de deletar permanentemente
        instance.is_active = False
        instance.save()
        # ou: instance.delete() para deletar de verdade
```

---

# 🔟 Exemplo Completo - Tudo Junto

```python
from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAdminUser, AllowAny
from rest_framework.pagination import PageNumberPagination
from django_filters.rest_framework import DjangoFilterBackend
from .models import Product
from .serializers import (
    ProductListSerializer,
    ProductDetailSerializer,
    ProductCreateUpdateSerializer
)

class ProductPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class ProductViewSet(viewsets.ModelViewSet):
    """
    ViewSet completo para gerenciamento de produtos
    
    Endpoints disponíveis:
    - GET /products/ - Lista produtos
    - POST /products/ - Cria produto (admin)
    - GET /products/{id}/ - Detalhe do produto
    - PUT /products/{id}/ - Atualiza produto (admin)
    - PATCH /products/{id}/ - Atualiza parcialmente (admin)
    - DELETE /products/{id}/ - Remove produto (admin)
    - GET /products/low_stock/ - Produtos com estoque baixo
    - POST /products/{id}/add_stock/ - Adiciona estoque (autenticado)
    """
    queryset = Product.objects.all()
    pagination_class = ProductPagination
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['stock', 'price']
    search_fields = ['name']
    ordering_fields = ['price', 'stock', 'created_at']
    ordering = ['-created_at']
    
    def get_serializer_class(self):
        if self.action == 'list':
            return ProductListSerializer
        elif self.action in ['create', 'update', 'partial_update']:
            return ProductCreateUpdateSerializer
        return ProductDetailSerializer
    
    def get_permissions(self):
        if self.action in ['list', 'retrieve']:
            permission_classes = [AllowAny]
        elif self.action in ['create', 'update', 'partial_update', 'destroy']:
            permission_classes = [IsAdminUser]
        else:
            permission_classes = [IsAuthenticated]
        return [permission() for permission in permission_classes]
    
    def get_queryset(self):
        queryset = Product.objects.all()
        
        # Filtro por estoque disponível
        in_stock = self.request.query_params.get('in_stock', None)
        if in_stock:
            queryset = queryset.filter(stock__gt=0)
        
        return queryset
    
    @action(detail=False, methods=['get'])
    def low_stock(self, request):
        """GET /products/low_stock/ - Produtos com estoque baixo"""
        products = Product.objects.filter(stock__lt=10)
        serializer = self.get_serializer(products, many=True)
        return Response({
            'count': products.count(),
            'products': serializer.data
        })
    
    @action(detail=True, methods=['post'])
    def add_stock(self, request, pk=None):
        """POST /products/{id}/add_stock/ - Adiciona estoque"""
        product = self.get_object()
        amount = request.data.get('amount', 0)
        
        if not isinstance(amount, (int, float)) or amount <= 0:
            return Response(
                {"error": "Quantidade deve ser positiva"},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        product.stock += int(amount)
        product.save()
        
        return Response({
            'message': 'Estoque atualizado',
            'current_stock': product.stock
        })
    
    def perform_create(self, serializer):
        """Hook antes de salvar criação"""
        if serializer.validated_data.get('stock', 0) < 0:
            raise ValidationError("Estoque não pode ser negativo")
        serializer.save()
    
    def perform_destroy(self, instance):
        """Soft delete"""
        instance.is_active = False
        instance.save()
```

---

# 📊 Comparação Resumida

| Característica | ViewSet | ModelViewSet |
|----------------|---------|--------------|
| **Código necessário** | Alto (implementar tudo) | Baixo (automático) |
| **Flexibilidade** | Máxima | Média-Alta |
| **CRUD automático** | ❌ Não | ✅ Sim |
| **Melhor para** | Lógica customizada | CRUD padrão |
| **Curva de aprendizado** | Maior | Menor |

---

# 🎯 Quando Usar Cada Um?

**Use `ModelViewSet` quando:**
- Precisa de CRUD completo padrão
- Quer código mínimo
- O modelo segue convenções REST

**Use `ViewSet` quando:**
- Precisa de total controle
- A lógica não segue padrões CRUD
- Quer implementar ações muito específicas

**Sobrescreva métodos quando:**
- `ModelViewSet` serve, mas precisa de validações extras
- Quer adicionar lógica antes/depois de salvar
- Precisa customizar respostas

---

**Versão:** 2.0 - Completa e Atualizada