# Django

## Simulando o cliente
Nós podemos criar um teste simulando o acesso do cliente a uma determinada requisição no servidor.
```
def test_recipe_home_view_returns_status_code_200_OK(self):
    response = self.client.get(reverse('recipes:home'))
    self.assertEqual(response.status.code, 200) 
```

De acordo com o testem o código do status de resposta deverá ser 200.