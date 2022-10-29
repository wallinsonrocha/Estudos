# Python

## Dicionário.
O dicionário é como uma lista que amarzena uma chave e um valor. Ele é bem semelhante a uma estrutura.

```
pessoa = {"nome":"Wallinson", "idade":22}
pessoa = dict(nome="Wallinson", idade=22)

pessoa["telefone"] = "3333-1234" # pessoa = {"nome":"Wallinson", "idade":22, "telefone":"3333-1234"}
```

Para acessar ou modificar o valor devemos utilizar a chave.
```
pessoa["nome"] # "Wallinson"
pessoa["nome"] = "Sarah"

pessoa["nome"] # "Sarah"
```

## Dicionário aninhados
É uma estrutura semelhante aos objetos. É o mais perto do real.
```
contatos = {
    "guilherme@email.com": {"nome": "Guilherme", "telefone": "70707070"},
    "lara@email.com": {"nome": "Lara", "telefone": "70707070"}
}

contatos["lara@email.com"]["nome"] # Lara
```

## Métodos

### {}.fromkeys
Permite criarmos chaves e, opcionalmente, valores. Para as chaves passamos uma lista e para o valor, passamos fora da lista.
```
dict.fromkeys(["nomes", "telefone"]) # {"nome": None, "telefone": None}

dict.fromkeys(["nomes", "telefone"], "vazio") # {"nome": "vazio", "telefone": "vazio"}
```

### {}.get
Permite fazer uma busca no dicionário verificando, primeiro, se a chave existe.
```
contatos.get("chave") # None caso não existe.
cotantos.get("chave", {}) # {} caso não haja objetos ou a chave não exista.
contato.get("lara@email.com", {}) # {"nome": "Lara", "telefone": "70707070"}
```

### {}.items
Retorna uma tipla dos elementos do dicionário. É bastante utilizado no for.
```
contatos = {
    "guilherme@email.com": {"nome": "Guilherme", "telefone": "70707070"}
}

contatos.items() # dict_items([("guilherme@email.com", {"nome": "Guilherme", "telefone": "70707070"})])
```

### {}.keys
Retornará as chaves em uma tupla, apenas.
```
contatos = {
    "guilherme@email.com": {"nome": "Guilherme", "telefone": "70707070"}
}

contatos.keys() # dict_keys(["guilherme@email.com"])
```

### {}.values
Fará o mesmo trabalho das chaves, porém será aplicada nos valores.

### {}.setdefault
Põe como padrão uma chave e um valor caso elas não existam.
```
contatos = {"nome": "Guilherme", "telefone": "70707070"}

contato.setdefault("nome", "Sarah") # Como o valor chave já existe, não será modificado nem adicionado.

contato.setdefault("idade", 22) # Como a chave não existe, ela será adicionada no dicionário.
```

### {}.update
Permite atualizar os valores de um dicionário.
```
contato.update({"guilherme@email.com": {"nome": "Gui"}}) # Nesse caso o nome será alterado para "Gui".
```

### del
Deletará toda a chave especificada.
```
del contatos["guilherme@email.com"]["telefone"] # Irá deletar a chave com o seu valor.
```

### {}.copy
Copia os valores de um dicionário em uma variável.

### {}.pop("chave")
Deleta chave especificada no dicionário.

### {}.popitem
É o pop tradicional.