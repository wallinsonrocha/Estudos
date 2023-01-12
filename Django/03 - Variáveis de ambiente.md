# Django

## Preparação
```
pip install python-dotenv
```

Devemos importar o **load_dotenv** nos seguintes arquivos:
```
from dotenv import load_dotenv
```

- `manage.py`
```
from dotenv import load_dotenv
...

def main():
    load_dotenv()
    ...
```
- `asgi.py` e `wsgi.py` em project
```
from dotenv import load_dotenv
...

load_dotenv() #acima do arquivo abaixo
 os.environ...
```

## Criar arquivo .env
Após isso devemos criar o arquivo `.env` e `.env_example`. O único que não irá para o github será o `.env`. Já o `.env_exemple` serve para orientar os outros desenvolvdores a criar um .env para a sua aplicação colocando alguns valores nas variáveis que estarão especificadas dentro do arquivo.

No .env normal colocaremos as variáveis e os seus valores para serem colocados na aplicação.

```
PER_PAGE = 9
```

> Todos os arquivos aqui, quando forem importados para outro arquivo, irão como String.

## Colocando-os na view
Para colocar na view de algum app, basta colocarmos os seguintes códigos.
```
import os
...
PER_PAGE = int(os.environ.get('PER_PAGE', 6)) # Chave e valor padrão
```

Principal parte do código: `os.environ.get('PER_PAGE', 6)`.

As modificações só serão feitas quando o servidor recarregar.

## Recomendações
Caso esejamos lidando com valores booleanos, podemos criar lógicas para exercer tal fução que queremos.

```
DEBUG = 1
```
```
DEBUG = True if os.environ.get('DEBUG', 'ISECURE') == '1' else False
```