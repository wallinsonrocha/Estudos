# Django

## Exposição e modificação de atributos
Em na nosso arquivo que colocamos os arquivos para serem reconhecidos no django admin, podemos dizer como eles serão expostos.
Exemplo:
```
@admin.register(Recipe)
class RecipeAdmin(admin.ModelAdmin):
    # Aqui modifica como as informações irão aparecer
    list_display = ['id', 'title', #...]
    # Adcionará link para ter acesso à publicação
    list_display_links = 'title', 'created_at',
    # Campo de busca
    search_fileds = 'id', 'title',
    # Filtro para melhorar a busca
    list_filter = 'category', 'author', 'is_published',
    # Quantos conteúdos aparecerão por página
    list_per_page = 10
    # O que podemos editar sem precisar entrar pelo link
    list_editable = 'is_published'
    # Forma de ordenação
    ordering = '-id'
```

> Não podemos esquecer que os valores são apenas exemplos. Eles são atributos do model principal que criamos.

## Automatização do slug
Para editar o slig autimaticamente.
```
@admin.register(Recipe)
class RecipeAdmin(admin.ModelAdmin):
    prepopulated_fields = {
        "slug": ('title',)
    }
```