# Django

## ORM
Para criarmos os nossos modelos relacionais, assim como o java, estabeleceremos-os em classes. E, como argumento, colocamos models.Model que irá servir como "validações" para os atributos. Ela é criada no arquivo **models** da nossa aplicação.
```
class Recipes(models.Model):
    title = models.CharField(max_length=65)
    description = models.CharField(max_length=165)
    slug = models.SlugField()
    preparation_time = models.IntegerField()
    preparation_time_unit = models.CharField(max_length=65)
    servings = models.IntegerField()
    servings_unit = models.CharField(max_length=65)
    preparation_steps = models.TextField()
    preparation_steps_is_html = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True) # Adiciona no momento da criação
    updated_at = models.DateTimeField(auto_now=True) # Para atualizar
    is_published = models.BooleanField(default=False)
    # imagem
    cover = models.ImageField(upload_to='recipes/covers/%Y/%m/%d/') # Caminho definindo dia, mês e ano automaticamente.

```

**Obs.:** No que tange à imagem, precisaremos modificar o arquivo **settings.py**. No final, para indicar onde as imagens estarão salvas, colocaremos
```
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

Além disso, precisamos configurar o arquivo **urls.py** do **projeto principal** (não o da aplicação criada). Lá iremos adicionar:
```
...
from django.conf.urls.static import static
from django.conf import settings
...
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

### __str__(self)
Podemos utilizar esse formato para que o nome seja retornado, ao invés de objeto. Colocamos em uma classe.
```
class Recipes(models.Model):
    name = models.CharField(max_length=65)
    ...
    def __str__(self):
        return self.name
```

## Relações
Supomos que um dos atributos depende de uma classe para ser preenchida. Para fazer a relação entre elas, a configuração é a seguinte:
#### Classe categoria
```
class Category(models.Model):
    name = models.CharField(max_length=65)
```
#### Classe recipes
```
class Recipes(models.Model):
    ...
    category = models.ForeignKey(
        Category, on_delete=models.SET_NULL, null=True, blank=True, default=None
    )
```

Dentro do foreignKey selecionamos para quem ele faz a relação, o que fazer quando for deletado e permitir estar nulo.

### Relações com Auth
Aqui devemos fazer a importação da classe User para poder manipular as relações do usuário.
```
from django.contrib.auth.models import User

class Recipes(models.Model):
    ...
    author = models.ForeignKey(
        User, on_delete=models.SET_NULL, null=True
    )
```

## Importações e migrações
Após fazer todos esse processo, caso estejamos trabalhando com imagens, devemos intalar o Pillow.
```
pip install pillow
```

Depois precisamos salvar para prepará-los para o banco de dados.
```
python manage.py migrate
```
Isso irá aplicar as migrações para serem salvas no banco de dados (semelhante ao git commit).

Após isso iremos fazer as migrações.
```
python manage.py makemigrations
```
Dessa forma os Models serão criados.

