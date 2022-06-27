# Aula 1

## Declaração de variáveis

Exsistem 3 formas de se declarar uma variável no kotlin.

1. **Var:** O valor da variável pode ser alterado durante o código.
2. **Val:** A variável terá somente o valor atribuído. Semelhante à constante, mas o seu valor pode ser definido depois.
3. **Const Val:** Constante cuja atribuição é feita durante a compilação. Ele, assim com o Val, não poderá ser mudado. Mas o seu valor deve ser colocado logo no início.

## Tipagem

Para definir o tipo de alguma variável, nós devemos faze-la como no Typescript. Além disso, geralmente, ela só é necessária quando o seu valor não é atribuído na criação.
```
var numero:Int
numero = 20
```

Os tipos são como o do Java.

## Tipo nulo
Para dizer que o valor de uma variável é nula, devemos colocar um "?" após a tipagem.
```
var numero:Int? = null;
```

## Operadores In e range
Com o in, nós podemos verificar se há ou não algo que procuramos. Ele pode ser usado como in(contém) e como !in(não contém). Por sua vez, o range cria um intervalo de números.
```
val numbers = lostOf(3, 9, 0, 1, 2);
print(12 in numbers);
//false

print(12 in 0..20); // in com ranger
//true
```

O range é usado até mesmo com o método random(). Observer:
```
var age = (10..100).random()
```

## Observações
Os operadores aritiméticos, Operadores de comparação e os operadores lógicos são semelhantes ao do javascript. Para isso, caso queira observar, você pode verificar nas minhas anotações sobre Operadores na seção de javascript.