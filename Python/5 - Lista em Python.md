# Python

## Trabalhando com listas em Python

Uma lista em python pode armazenar qualquer tipo de objeto. Nós a criamos utilizando o construtor **list**, a função range ou colocando valores separados por vírgulas dentro de colchetes.

```
frutas = ["laranja", "maçã", "uva"]

frutas = [] // Lista vazia

letras = list("python")

numeros = list(range(10))

carro = ["Ferrari", "F8", 123, True]
```

## Acessando elementos

Para acessar um elemento da lista, devemos especificar o seu índice da seguinte forma:

```
frutas = ["laranja", "maçã", "uva"]

frutas[0] // laranja
frutas[1] // maçã
frutas[-1] // uva
```

A sua contagem inicia a partir do número 0. Caso escrevamos um número negativo como -1, ele irá contar ao contrário.

## Matrizes

São listas que contém outras listas dentro.

```
matriz = [
    [1, "a", 2],
    ["b", 3, 4],
    [6, 5, "c"]
]

matriz[0] // [1, "a", 2]
matriz[0][0] // 1
matriz[0][-1] // 2
matriz[2][-1] // c
```

## Fatiamento

É bem semelhante ao fatiamento de uma string.

```
letras = list("python")

letras[2:] // 't', 'h', 'o', 'n'
letras[:2] // 'p', 'y'
letras[1:3] // 'y', 't'
letras[0:3:2] // 'p', 't'
```

## Percorrer uma lista

Podemos usar o comando for para percorrer a lista.

```
carros = ["gol", "celta", "palio"]

for carro in carros:
    print(carro)
```

## Função enumerate

Às vezes é necessário saber qual o índice do objeto dentro do laço for. Para isso podemos usar a função enumerate.

```
carros = ["gol", "celta", "palio"]

for indice, carro in enumerate(carros)
    print(f"{indice}: {carro}")

// 0: gol
// 1: celta
// 2: palio

```
