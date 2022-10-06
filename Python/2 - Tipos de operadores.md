# Python

## Tipos de operadores

### Aritméticos
Os operadores aritiméticos são utilizados para realizar cálculos matemáticos.

Operador | Utilização | Exemplo
-------- | ---------- | -------
% | Módulo. Serve para retornar o valor do resto da divisão. Muito utilizado para verificar se um número é par ou ímpar. | 4 % 2 retornará 0
** | Exponenciação. Ele irá realizar um cálculo exponencial. O primeiro número é a base e o segundo o expoente. | 3 ** 2 = 8
// | Divisão inteira. Ele irá realizar uma divisão, porém, retornará um número inteiro. | 12 // 5 = 2

Há outros operadores, Mas os mais utilizados são esses. Não podemos esquecer também que, em uma operação matemática simples, podemos fazer normalmente assim:
```
x = (2+3) * 5;
y = (4-2) / x;
```

### Atribuição.
Os operadores de atribuição serve para atribuir um valor a alguma variável.

Operador | Utilização | Significado
-------- | ---------- | -----------
= | Utilizado para dizer que a variável irá ter o valor definido por nós. | x = y
+= | Geralmente usado em cálculos matemáticos, esse operador irá somar o valor da variável mais o valor que colocamos. | x = x + y
-= | A utilização é semelhante ao operador anterior, mas, ao invés de somar, irá subtrair. | x = x - y
*= | Como os exemplos acima, mas irá multiplicar o valor. | x = x * y
/= | Como os exemplos acima, mas irá dividir. | x = x / y
%= | Como os exemplos acima, mas irá coletar o resto da divisão. | x = x % y
**= | Como os exemplos acima, mas irá elevar ao valor escolhido. | x = x ** y

### Comparação.
Os operadores de comparação são muito utilizados para verificar condições.

Operador | Utilização | Exemplo
-------- | ---------- | -------
== | Utilizado para verificar igualdade. Ele é diferente de "=" pois não atribui valor algum. | 3 == 3
!= | Serve para verificar se é diferente. | 3 != 2
\>  | Símbolo de maior que. | 3 > 2
\>=  | Símbolo de maior ou igual que. | 3 >= 2 ou 3 >= 3
<  | Símbolo de menor que. | 2 < 3
<=  | Símbolo de menor ou igual que. | 2 <= 3 ou 2 <= 2

### Lógico
Utilizados muito para estabelecer condições.

Operador | Utilização | Exemplo
-------- | ---------- | -------
and | E. Utilizado para dizer que uma condição e outra devem ser atendidas para continuar. | x == 3 and y != 4
or | Ou. Utilizados para dizer que uma ou outra condição deve ser atendida. | x == 2 or y != 2
not | Não. Utilizado para retornar o valor contrário de um operador lógico. | not a > b Irá retornar o valor contrário dependendo dos valores de a e b. Se forem, respectivamente 2 e 5, o resultado será True.

Esses são os mais utilizados.

### Identidade
São operadores utilizados para comparar se dois objetos ocupam a mesma posição na memória.

Operador | Utilização | Exemplo
-------- | ---------- | -------
is | Está. É muito semelhante ao "==". Pode ser usado juntamento com o not: "is not". | x is y Se os valores forem, respectivamente, 3 e 3, irá retornar True.

### Associação
São operadores utilizados para verificar se há algum valor em uma sequência.

Operador | Utilização | Exemplo
-------- | ---------- | -------
in | Em. Ele verifica se valor a está em b. Também pode ser usado com o not: "not in". | Sendo x = ["laranja", "uva", "limão"]. "laranja" in x retornará True.
