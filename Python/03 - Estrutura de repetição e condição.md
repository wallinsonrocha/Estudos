# Python

## Estrutura de condição.

Para isso, usamos o if, else ou o elif.

```
x = int(input("Digite um número"))

if x == 30:
    print("x É 30")
elfi x == 20:
    print("x é 20")
else:
    print("x é um número diferente de 20 e 30")
```

Podemos armazená-lo em uma variável. A sua estrutura é:

```
variável = <resultado> if <condição> else <OutroResultado>
// Exemplo
status = "Sucesso" if saldo >= saque else "Falha"
```

## Estrutura de repetição

Usado para repetir um escopo várias vezes.

### for
O for no python é um foreach. Ele irá levar em consideração um parâmetro (variável ou lista) para repetir.

```
texto = input("Informe um texto")
VOGAIS = "AEIOU"

for letra in texto:
    if letra.upper() in VOGAIS:
        print(letras, end="")
```

Verifique que a variável "letra" não existe fora do escopo do for. Isso porque serve apenas para aquele momento.

Nós podemos usar, também, o rangem que irá criar uma lista de números.
```
// Estrutura
range(inicio, fim, intervalo)

// Exemplo

for numeros in range(0, 50, 5):
    print(numero, end=" ")
```

Nesse caso iremos criar uma tabuáda de 5.

Obs.: No range o único parâmetro obrigatório é o fim.

### while

Com while temos um for classico. Ele irá repetir até uma determinada condição.

```
opcao = -1

while opcao != 0:
    opcao = int(input("digite um número: "))
```

Nesse caso o laço só irá terminar se digitarmos 0. Mas nós podemos fazer uma quebra forçada com break, também:
```
opcao = -1

while opcao != 0:
    opcao = int(input("digite um número: "))
    if opcao == 1:
        break
```

O contrário de break é o continue.