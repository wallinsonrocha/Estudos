# Python

## [].append()

Sua principal função é adicionar algum elemento a alguma lista.

```
lista = []

lista.append(1)
lista.append("Python")

print(lista) // [1, "Python"]
```


## [].extend()

Este, direfente do append, pode adicionar vários elementos dentro de uma lista.

```
ling = ["python", "js", "c"]

ling.extend(["java", "csharp"])

print(ling) // ["python", "js", "c", "java", "csharp"]
```


## [].copy()

Copia uma lista para outra.

```
lista1 = [1, 2, 3, 4]
lista2 = lista1.copy()

print(lista2) // 1, 2, 3, 4
```


## [].count()

Serve para contar a quantas vezes um certo elemento é repetido.

```
numeros = [1, 1, 1, 2, 3, 4, 5, 6]

numeros.count(1) // 3
```


## [].index()

Para saber a localização de um elemento.

```
ling = ["python", "js", "c", "java", "csharp"]

ling.index("java") // 3
```


## [].pop()

Para remover elementos. Por padrão ele remove o último elemento. Porém, podemos modificá-lo colocando o índice. 

```
ling = ["python", "js", "c", "java", "csharp"]

ling.pop() // ["python", "js", "c", "java"]
ling.pop(0) // ["js", "c", "java"]
```


## [].remove()

Diferente do pop, neste você especifica o elemento.

```
ling = ["python", "js", "c", "java", "csharp"]

ling.remove("c") // ["python", "js", "java", "csharp"]
```


## [].reverse()

Inverte os índices.

```
ling = ["python", "js", "c", "java", "csharp"]

ling.reverse() // ['csharp', 'java', 'c', 'js', 'python']
```


## [].sort()

Ordenará, de forma padrão, a lista em ordem crescente ou alfabética. Mas podemos colocar alguns parâmtros para indicar a sua ordem.

```
ling.sort() // ['c', 'csharp', 'java', 'js', 'python']
ling.sort(reverse=True) // ['python', 'js', 'java', 'csharp', 'c']

// Por tamanho
ling.sort(key=lambda x: len(x)) // ['c', 'js', 'java', 'python', 'csharp']
ling.sort(key=lambda x: len(x), reverse=True) // ['python', 'csharp', 'java', 'js', 'c']
```