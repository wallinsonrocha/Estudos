# Aula 2

> Obs, alguns operadores estão com entre " ". Isso foi feito para evitar o conflito de código do markdown. O mesmo teve que ser feito com o " || " , pois, para que ele aparecesse escrevi sua simbologia literalmente "Barra dupla vertical". Em breve isso será corrigido.

Os operadores, no javascript e outras linguagens de programação podem exercer diversas funções. geralmente eles são usados para auxiliar nas operações matemática ou estabelecer condições em um laço de repetição ou de comparação.

## Operadores de atribuição.
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

Há outros exemplos, mas os mais utilizados são esses.

## Operadores de comparação.
Os operadores de comparação são muito utilizados para verificar condições.

Operador | Utilização | Exemplo
-------- | ---------- | -------
== | Utilizado para verificar igualdade. Ele é diferente de "=" pois não atribui valor algum. | 3 == 3
!= | Serve para verificar se é diferente. | 3 != 2
=== | Este é semelhante ao "==" porém, ele verifica se o tipo da variável também é igual. Para que a condição seja verdadeira, não podemos ter algo como '3' === 3 | '2' === '2'
" > " | Símbolo de maior que. | 3 > 2
" >= " | Símbolo de maior ou igual que. | 3 >= 2 ou 3 >= 3
" < " | Símbolo de menor que. | 2 < 3
" <= " | Símbolo de menor ou igual que. | 2 <= 3 ou 2 <= 2

## Operadores aritiméticos.
Utilizados em operações matemáticas.

Operador | Utilização | Exemplo
-------- | ---------- | -------
% | Módulo. Serve para retornar o valor do resto da divisão. Muito utilizado para verificar se um número é par ou ímpar. | 4 % 2 retornará 0
++ | Incremento. Ele somar mais 1 unidade. | x++ é a mesma que x += 1 que é x = x + 1
-- | Decremento. Ele irá diminuir 1 unidade | x-- é a mesma que x -= 1 que é x = x - 1

Há outros operadores, Mas os mais utilizados são esses. Não podemos esquecer também que, em uma operação matemática simples, podemos fazer normalmente assim, também:
```
x = (2+3) * 5;
y = (4-2) / x;
```

## Operadores bit a bit.
Utilizados muito para estabelecer condições.

Operador | Utilização | Exemplo
-------- | ---------- | -------
&& | E. Utilizado para dizer que uma condição e outra devem ser atendidas para continuar. | x == 3 && y != 4
Barra dupla vertical | Ou. Utilizados para dizer que uma ou outra condição deve ser atendida. | x == 2 Barra dupla vertical y != 2

Esses são os mais utilizados.
