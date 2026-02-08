# Guia Completo: Django REST Framework Views

## 1. **`APIView`**

Essa é a classe base do DRF para criar views. Ela funciona de forma similar à `View` do Django, mas já entende requisições e respostas em JSON.

**Quando usar:**

* Quando você quer controle total sobre cada método (`get`, `post`, `put`, `delete`).
* Quando não precisa de abstrações automáticas como listagem ou CRUD.
* Para lógica de negócio muito específica que não se encaixa nos padrões.

**Exemplo:**

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import IsAuthenticated

class HelloWorldAPIView(APIView):
    permission_classes = [IsAuthenticated]  # Controle de permissões
    
    def get(self, request):
        return Response({"message": "Hello, World!"}, status=status.HTTP_200_OK)

    def post(self, request):
        name = request.data.get("name")
        if not name:
            return Response(
                {"error": "Nome é obrigatório"}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        return Response({"message": f"Hello, {name}"}, status=status.HTTP_201_CREATED)
```

**Observação:** você precisa tratar manualmente coisas como validação, querysets e filtragem.

**Vantagens:**
* Controle total sobre a lógica
* Flexibilidade máxima

**Desvantagens:**
* Mais código para escrever
* Precisa implementar tudo manualmente

---

## 2. **`GenericAPIView` + Mixins**

Essa é a base para views mais inteligentes. Ela já fornece:

* Acesso a `queryset` e `serializer_class`.
* Métodos de filtro, busca e paginação.

Os **mixins** adicionam comportamento de CRUD automaticamente fornecendo métodos auxiliares:

* `CreateModelMixin` → fornece `create()`
* `ListModelMixin` → fornece `list()`
* `RetrieveModelMixin` → fornece `retrieve()`
* `UpdateModelMixin` → fornece `update()` e `partial_update()`
* `DestroyModelMixin` → fornece `destroy()`

**Importante:** Os mixins fornecem métodos auxiliares que você conecta aos métodos HTTP.

**Exemplo:**

```python
from rest_framework import generics, mixins
from myapp.models import Product
from myapp.serializers import ProductSerializer

class ProductListCreateView(mixins.ListModelMixin,
                            mixins.CreateModelMixin,
                            generics.GenericAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)  # Chama o método do mixin

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)  # Chama o método do mixin
```

**Pontos importantes:**

* Misturar mixins é comum (`ListModelMixin + CreateModelMixin`).
* Você ainda precisa definir manualmente os métodos HTTP (`get`, `post`, etc.).
* Os mixins cuidam da lógica interna (validação, salvamento, etc.).

**Vantagens:**
* Menos código que APIView
* Reutilização de lógica comum

**Desvantagens:**
* Ainda precisa mapear métodos HTTP manualmente

---

## 3. **`ListCreateAPIView` e `RetrieveUpdateDestroyAPIView`**

Essas classes combinam `GenericAPIView + Mixins` para casos comuns de CRUD. São **pré-configuradas** e economizam código.

**Exemplos:**

**Listar e criar produtos:**

```python
from rest_framework import generics
from myapp.models import Product
from myapp.serializers import ProductSerializer

class ProductListCreateView(generics.ListCreateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    
    # Opcional: adicionar filtros, paginação, etc.
    filterset_fields = ['category', 'price']
```

**Recuperar, atualizar ou deletar um produto específico:**

```python
class ProductDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    lookup_field = 'slug'  # Padrão é 'pk', mas pode customizar
```

**Classes disponíveis:**

* `CreateAPIView` → apenas POST
* `ListAPIView` → apenas GET (lista)
* `RetrieveAPIView` → apenas GET (detalhe)
* `DestroyAPIView` → apenas DELETE
* `UpdateAPIView` → PUT e PATCH
* `ListCreateAPIView` → GET (lista) e POST
* `RetrieveUpdateAPIView` → GET (detalhe), PUT e PATCH
* `RetrieveDestroyAPIView` → GET (detalhe) e DELETE
* `RetrieveUpdateDestroyAPIView` → GET (detalhe), PUT, PATCH e DELETE

**Vantagens:**

* Sem precisar escrever `get`, `post`, `put`, `delete`.
* DRF já cuida de validação e retorno de status.
* Código muito limpo e direto.

**Desvantagens:**
* Menos flexível para lógica customizada
* Pode ser necessário sobrescrever métodos para casos específicos

---

## 4. **`ViewSet` e `ModelViewSet`**

Agora a coisa fica profissional. Se você quer **menos código e URLs automáticas**, use ViewSets.

**`ViewSet`** → você define ações (`list`, `retrieve`, `create`) manualmente.
**`ModelViewSet`** → é o CRUD completo automático. Você só precisa definir `queryset` e `serializer_class`.

**Exemplo de ModelViewSet:**

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response
from myapp.models import Product
from myapp.serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    lookup_field = 'slug'  # Customizar campo de busca (padrão é 'pk')
    
    # Ação customizada
    @action(detail=False, methods=['get'])
    def recent(self, request):
        """Retorna os 5 produtos mais recentes"""
        recent = Product.objects.all().order_by('-created_at')[:5]
        serializer = self.get_serializer(recent, many=True)
        return Response(serializer.data)
    
    @action(detail=True, methods=['post'])
    def add_stock(self, request, pk=None):
        """Adiciona estoque a um produto específico"""
        product = self.get_object()
        amount = request.data.get('amount', 0)
        product.stock += amount
        product.save()
        return Response({'status': 'stock updated', 'current_stock': product.stock})
```

Depois você conecta à URL usando um **router**, que cria todas as rotas automaticamente:

```python
from rest_framework.routers import DefaultRouter
from django.urls import path, include
from myapp.views import ProductViewSet

router = DefaultRouter()
router.register(r'products', ProductViewSet, basename='product')

urlpatterns = [
    path('api/', include(router.urls)),
]
```

**Resultado:**

* `GET /api/products/` → lista todos os produtos
* `POST /api/products/` → cria um novo produto
* `GET /api/products/{id}/` → detalhe de um produto
* `PUT /api/products/{id}/` → atualiza completamente
* `PATCH /api/products/{id}/` → atualiza parcialmente
* `DELETE /api/products/{id}/` → deleta
* `GET /api/products/recent/` → ação customizada (lista produtos recentes)
* `POST /api/products/{id}/add_stock/` → ação customizada (adiciona estoque)

**Tipos de ViewSets:**

* `ViewSet` → base, você implementa tudo
* `GenericViewSet` → base + GenericAPIView (sem ações padrão)
* `ModelViewSet` → CRUD completo (list, create, retrieve, update, destroy)
* `ReadOnlyModelViewSet` → apenas list e retrieve

**Vantagens:**
* URLs automáticas via router
* Código extremamente conciso
* Ações customizadas com `@action`
* Agrupa lógica relacionada em uma classe

**Desvantagens:**
* Menos intuitivo para iniciantes
* Mais abstrato, pode dificultar debug

---

## 5. **`Serializers`**

Não dá pra falar de DRF sem serializer. Ele transforma **modelos em JSON** e **valida dados**.

**Exemplo básico:**

```python
from rest_framework import serializers
from myapp.models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'stock', 'category']
        # Ou use '__all__' para incluir todos os campos
        # fields = '__all__'
        
        # Campos somente leitura
        read_only_fields = ['id', 'created_at']
```

**Validação customizada:**

```python
class ProductSerializer(serializers.ModelSerializer):
    # Campo calculado (não existe no modelo)
    total_value = serializers.SerializerMethodField()
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'stock', 'category', 'total_value']
    
    def get_total_value(self, obj):
        """Calcula valor total em estoque"""
        return obj.price * obj.stock
    
    def validate_price(self, value):
        """Validação de campo específico"""
        if value <= 0:
            raise serializers.ValidationError("Preço deve ser positivo")
        return value
    
    def validate(self, data):
        """Validação que envolve múltiplos campos"""
        if data.get('stock', 0) < 0:
            raise serializers.ValidationError("Estoque não pode ser negativo")
        
        if data.get('price', 0) > 10000 and data.get('stock', 0) > 100:
            raise serializers.ValidationError(
                "Produtos caros não podem ter estoque tão alto"
            )
        
        return data
```

**Serializers aninhados:**

```python
class CategorySerializer(serializers.ModelSerializer):
    class Meta:
        model = Category
        fields = ['id', 'name']

class ProductSerializer(serializers.ModelSerializer):
    category = CategorySerializer(read_only=True)  # Serializer aninhado
    category_id = serializers.IntegerField(write_only=True)  # Para escrita
    
    class Meta:
        model = Product
        fields = ['id', 'name', 'price', 'category', 'category_id']
```

**Dicas práticas:**

* Use `ModelSerializer` para modelos Django, economiza tempo.
* Use `Serializer` puro se for um JSON customizado, sem modelo.
* Validação customizada com `validate_<field>` para um campo específico.
* Use `validate()` para validações que envolvem múltiplos campos.
* `SerializerMethodField` para campos calculados.

---

## 6. **Paginação**

Controle quantos resultados retornar por página:

```python
from rest_framework.pagination import PageNumberPagination

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    pagination_class = StandardResultsSetPagination
```

**Tipos de paginação:**
* `PageNumberPagination` → `/products/?page=2`
* `LimitOffsetPagination` → `/products/?limit=10&offset=20`
* `CursorPagination` → para grandes datasets, mais eficiente

---

## 7. **Filtros e Busca**

```python
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter, OrderingFilter

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    
    # Filtros exatos
    filterset_fields = ['category', 'price']
    
    # Busca por texto
    search_fields = ['name', 'description']
    
    # Ordenação
    ordering_fields = ['price', 'created_at']
    ordering = ['-created_at']  # Ordenação padrão
```

**Uso:**
* `/products/?category=1` → filtra por categoria
* `/products/?search=laptop` → busca por "laptop"
* `/products/?ordering=-price` → ordena por preço decrescente

---

## 8. **Permissões**

```python
from rest_framework.permissions import IsAuthenticated, IsAdminUser

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
    
    def get_permissions(self):
        """Permissões diferentes por ação"""
        if self.action in ['create', 'update', 'destroy']:
            return [IsAdminUser()]
        return [IsAuthenticated()]
```

**Permissões comuns:**
* `AllowAny` → acesso livre
* `IsAuthenticated` → usuário autenticado
* `IsAdminUser` → apenas admins
* `IsAuthenticatedOrReadOnly` → leitura livre, escrita autenticada

---

💡 **Resumo rápido de uso real:**

| Classe | Quando usar | Nível de código | Flexibilidade | URLs |
|--------|-------------|-----------------|---------------|------|
| `APIView` | Total controle, lógica customizada | Alto | Máxima | Manual |
| `GenericAPIView + Mixins` | CRUD parcial com controle | Médio | Alta | Manual |
| `ListCreateAPIView / RetrieveUpdateDestroyAPIView` | CRUD padrão, casos comuns | Baixo | Média | Manual |
| `ViewSet` | Agrupar ações relacionadas | Médio | Alta | Router |
| `ModelViewSet` | CRUD completo automático | Muito baixo | Média-Baixa | Router |

---

## 🎯 **Fluxo de decisão: Qual usar?**

```
Precisa de lógica muito específica/customizada?
  └─ SIM → APIView
  └─ NÃO ↓

É um CRUD padrão de um modelo?
  └─ SIM → ModelViewSet (mais rápido)
  └─ NÃO ↓

Precisa de apenas algumas operações CRUD?
  └─ SIM → ListCreateAPIView, RetrieveUpdateDestroyAPIView, etc.
  └─ NÃO ↓

Quer agrupar várias ações relacionadas?
  └─ SIM → ViewSet customizado
  └─ NÃO → GenericAPIView + Mixins
```

---

## 📚 **Boas práticas:**

1. **Começar simples:** ModelViewSet para CRUD básico, depois customizar se necessário
2. **Serializers separados:** Um para leitura, outro para escrita quando necessário
3. **Validação no serializer:** Não no modelo ou view
4. **Permissões granulares:** Use `get_permissions()` para controle fino
5. **Documentação:** Use docstrings, facilita geração de documentação automática
6. **Versionamento:** Planeje desde o início (`/api/v1/products/`)
7. **Testes:** Sempre! DRF tem ótimas ferramentas para testes de API

---

**Ferramentas úteis:**
* `django-filter` → filtros avançados
* `drf-spectacular` → documentação OpenAPI/Swagger automática
* `djangorestframework-simplejwt` → autenticação JWT
* `drf-nested-routers` → rotas aninhadas para relacionamentos

---

**Versão:** 2.0 - Atualizada e expandida
**Mantido por:** [Seu Nome]