# Django

## Criação
Esses objetos irão nos ajudar no teste da criação. Irá poupar fazer isso manualmente sempre que tentarmos testar uma função que precisa deles.
```
from recipes.models import Category, Recipe, User
...

def test_recipe_home_template_loads_recipes(self):
    category = Category.objects.create(
        name='Category'
    )
    author = User.objects.create_user(
        first_name='user',
        last_name='name',
        username='username',
        password='123456',
        email='username@email.com',
    )
    recipe = Recipe.objects.create(
        category=category,
        author=author,
        title='Recipe Title',
        description='Recipe Description',
        slug='recipe-slug',
        preparation_time=10,
        preparation_time_unit='Minutos',
        servings=5,
        servings_unit='Porções',
        preparation_steps='Recipe Preparation Steps',
        preparation_steps_is_html=False,
        is_published=True,
    )

    response = self.client.get(reverse('recipes:home'))
    self.assertEqual(len(response.context['recipes']), 1)
```

Para isso basta importar os **models** para poder ter acesso às funções. No exemplo acima verificamos se a quantidade criada é igual a 1. Mas, além dele, podemos fazer vários outros testes. Podemos utilizar o context e o content.

Um exemplo com **content**.
```
def test_recipe_home_template_loads_recipes(self):
    category = Category.objects.create(
        name='Category'
    )
    author = User.objects.create_user(
        first_name='user',
        last_name='name',
        username='username',
        password='123456',
        email='username@email.com',
    )
    recipe = Recipe.objects.create(
        category=category,
        author=author,
        title='Recipe Title',
        description='Recipe Description',
        slug='recipe-slug',
        preparation_time=10,
        preparation_time_unit='Minutos',
        servings=5,
        servings_unit='Porções',
        preparation_steps='Recipe Preparation Steps',
        preparation_steps_is_html=False,
        is_published=True,
    )    

    response = self.client.get(reverse('recipes:home'))
    content = response.content.decode('utf-8') # Transforma o html em uma string para ser analisado.
    response_context_recipes = response.context['recipes']

    self.assertIn('Recipe Title', content)
    self.assertIn('10 Minutos', content)
    self.assertIn('5 Porções', content)
    self.assertEqual(len(response_context_recipes), 1)
```

Nesse caso verificamos se, dentro do conteúdo, há palavras como 'Recipe Title'.

