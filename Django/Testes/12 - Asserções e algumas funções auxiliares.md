# Django

## reverse
Através dele podemos armazenar, em uma variável, uma url.
```
class RecipeURLsTest(TestCase):
    def test_recipe_home_url_is_correct(self):
        home_url = reverse('recipes:home')
        self.assertEqual(home_url, '/')
```
home_url, no caso, procura pela url de nome **home**.

```
 def test_recipe_category_url_is_correct(self):
        url = reverse('recipes:category', kwargs={'category_id': 1})
        self.assertEqual(url, '/recipes/category/1/')
```
Neste acima procuramos pela url de nome **category** onde o category_id é igual a 1.

Ele poderá recer tanto **args** que receberá uma tupla
```
url = reverse('recipes:category', args=(1,))
```
como também poderá receber kwargs
```
url = reverse('recipes:category', kwargs={'category_id': 1})
```

## resolve
Utilizada para verificar a view, ou seja, a renderização.
```
class RecipeViewsTest(TestCase):
    def test_recipe_home_view_function_is_correct(self):
        view = resolve('/')
        self.assertTrue(True)
```

## assert
As asserções servem para verificar o retorno dos nossos testes. Temos vários tipos de asserts.

### assertEqual
```
class RecipeURLsTest(TestCase):
    def test_recipe_home_url_is_correct(self):
        home_url = reverse('recipes:home')
        self.assertEqual(home_url, '/')
```
Aqui, por exemplo, verifica de a url da home é '/'. Além dissom o **reverse** faz a procura da nossa url.

### assertTrue
```
class RecipeViewsTest(TestCase):
    def test_recipe_home_view_function_is_correct(self):
        view = resolve('/')
        self.assertTrue(True)
```

### assertIs
```
...
from recipes import views
...

class RecipeViewsTest(TestCase):
    def test_recipe_home_view_function_is_correct(self):
        view = resolve(reverse('recipes:home'))
        self.assertIs(view.func, views.home)
```
Verifica identidades. Nesse caso ele vê se **view.func** é igual a **views.home** que foi importada.

```
def test_recipe_detail_view_function_is_correct(self):
    view = resolve(
        reverse('recipes:recipe', kwargs={'id': 1})
    )
    self.assertIs(view.func, views.recipe)
```
Outro exemplo.