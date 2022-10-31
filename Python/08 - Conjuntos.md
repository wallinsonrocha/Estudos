# Python

## Conjuntos
O conjunto é semelhante a uma lista. Porém, o seu diferencial está no armazenamento: não permite informações repetidas.

```
set([1, 2, 3, 1, 3, 4]) # 1, 2, 3, 4
set("abacaxi") # b, a, c, x, i
set(("palio", "gol", "celta", "palio")) #"palio", "gol", "celta"
```

Para acessar os elementos de um conjunto, nós deveremos, primeiramente, convertê-lo em uma lista.
```
numeros = {1, 2, 3, 1, 3, 4}
numeros = list(numeros)
numeros[0] #1
```

## Métodos

### {}.union
```
conjuntoA = {1, 2}
conjuntoB = {3, 4}

conjuntoA.union(conjuntoB) # {1, 2, 3, 4}
```

### {}.intersection
```
conjuntoA = {1, 2, 3}
conjuntoB = {2, 3, 4}

conjuntoA.intersection(conjuntoB) # {2, 3}
```

### {}.difference
```
conjuntoA = {1, 2, 3}
conjuntoB = {2, 3, 4}

conjuntoA.difference(conjuntoB) # {1}
conjuntoB.difference(conjuntoA) # {4}
```

### {}.symmetric_difference
```
conjuntoA = {1, 2, 3}
conjuntoB = {2, 3, 4}

conjuntoA.symmetric_difference(conjuntoB) # {1, 4}
```

### {}.issubset
Verifica se um é subconjunto do outro.

```
conjuntoA = {1, 2, 3}
conjuntoB = {4, 1, 2, 5, 6, 3}

conjuntoA.issubset(conjuntoB) # True
conjuntoB.issubset(conjuntoA) # False
```

### {}.issuperset
Verifica se um conjunto está em outro.

```
conjuntoA = {1, 2, 3}
conjuntoB = {4, 1, 2, 5, 6, 3}

conjuntoA.issuperset(conjuntoB) # False
conjuntoB.issuperset(conjuntoA) # True
```

### {}.isdisjoint
Verifica se todos os elementos de um conjunto é diferente do outro.

```
conjuntoA = {1, 2, 3, 4, 5}
conjuntoB = {6, 7, 8, 9}
conjuntoC = {1, 0}

conjuntoA.isdisjoint(conjuntoB) # True
conjuntoB.isdisjoint(conjuntoA) # False
```

## Métodos conhecidos
### Add
Adicionar elementos.
### Clear
Limpar conjunto.
### Copy
Copia os valores em uma variável.
### Discard
Descartar valor escolhido.
### Pop
Excluir.
### Remove
Remover elemento selecionado.
### Len
Tamanho do conjunto escolhido.