# Django

## __init__.py
Há uma maneira de manipular o arquivo form para separar em arquivos diferentes e ser reconhecido pelo Django. No arquivo mencionado acima podemos colocar algumas importações. 

```
from .nome_do_arquivo import Class
from .nome_do_arquivo2 import Class2
```

> Não esqueça: Se criamos um arquivo `__init__.py`, obviamente ele está dentro de uma pasta. Ou seja, criar um pacote.