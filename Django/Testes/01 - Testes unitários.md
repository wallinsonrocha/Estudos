# Django

## Configurações iniciais
Para usar os testes precisamos instalar o **pytest** no nosso ambiente de desenvolvimento.
```
pip install pytest pytest-django
```

Após isso, precisamos criar um arquivo `pytest.ini` para que as configurações dos nossos testes sejam reconhecidas. 
```
[pytest]
DJANGO_SETTINGS_MODULE = project.settings # Nome do nosso projeto
python_files = test.py tests.py test_*.py tests_*.py *_test.py *_tests.py # Arquivos para serem reconhecidos
addopts = 
  --doctest-modules
  --strict-markers
  -rP
markers =
  slow: Run tests that are slow
  fast: Run fast tests
```

## Iniciando testes
Na pasta da nossa aplicação iremos até o arquivo de `tests.py`. Lá dentro iremos criar os nossos testes.
> Recomenda-se seguir a dica de criar uma pasta para colocar os testes.

Para criar, iremos usar uma classe que irá englobar todos os nossos testes unitários.
Exemplo.:
```
from django.test import TestCase

# Create your tests here.

class RecipeURLsTest(TestCase):
    def test_the_pytest_is_ok(self):
        assert 1 == 1, 'Um é igual a um'
```

> **assert** é um retorno obrigatório sobre um determinado resultado. Caso não seja retornado a condição que atribuímos, significa que estamos falhando no teste. Nele podemos colocar a condição que deve retornar e a mensagem (esta é opcional).

## Rodando nossos testes
```
pytest
```

## Dicas
Preferêncialmente é bom separarmos as nossas classes por pastas.

### Separar testes por pastas
Criaremos uma pasta chamada `tests` para armazenar nossas classes de testes.
> Não podemos nos esquecer de criar um arquivo `__init__.py`

Após isso, todo arquivo criado, por garantia, poderá começar com `test_`. Além disso, podemos excluir o arquivo `tests.py`.

Após isso, vale ressaltar que os arquivos criados para testes devem ter a importação do TestCase.
