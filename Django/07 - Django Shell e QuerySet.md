# Django 

O Shell é um programa que possibilita à usuária acessar recursos do Sistema Operacional, através do Terminal ou da interface gráfica. A diferença é que ele será utilizado para o Django utilizando Python.

QuerySet é um conjunto de ações que serão realizadas no banco de dados, ou seja, podemos criar, buscar, atualizar ou deletar os dados sem escrever a query SQL será executada no banco.

## Ativação:
```
python manage.py shell
```

## Importar e selecionar para manipular
```
from recipes.models import Recipe, Category

categories = Category.objects.all()
recipes = Recipe.objects.all()

# Irá retornar uma QuerySet com o objetos criados dentro de category

```

## Algumas bases
### MANIPULAÇÕES DE ORDEM
```
category.order_by('id') 
category.order_by('-id') # Ordenar por id descrescente
category.order_by('name')
category.order_by('-name')
category.order_by('id', '-name') 
# Manipulação com for
for recipe in recipes.order_by('id'): print(recipe.id, recipe.title)
```

Para criar um novo dado, recomenda-se utilizar um model que não **contenha foreing key** primeiro.

### CRIANDO DADO NO BANCO DE DADOS
```

new_category = Category()
new_category.name = 'nova categoria'
new_category.save() # Salva no banco de dados

#Outra forma é
new_category = Category.objects.create(name = 'nova categoria')
```

### ATUALIZANDO
```
dado = Category.objects.get(id=10)
dado.name = 'nome atualizado'
dado.save()
```

```
# DELETE
dado.delete()
```

### CRIANDO USUÁRIO
```
from django.contrib.auth.models impor User

User.objects.creat_user(first_name='Maria', last_name='Helena', username='mariahelena', email='maria@email.com', password='123456')
```

Ele é diferente dos demais pois a senha precisa ser criptografada.