# Aula 3

## Condicionais
### if
O if, que significa "se", estabelece uma condição. Todo if poderá se acompanhado ou não de um else (que será falado logo abaixo).

Padrão:
```
if(x >= 2){
    console.log('X é ' + x);
}
```

No caso acima houve apenas uma linha de código a ser executada. Nesse caso podemos escrever em apenas uma linha:
```
if(x >= 2) console.log('X é ' + x);
```

### else
O else, que significa "caso contrário", vem somente após um if ou caso tenha um else if, no final deste (que será visto mais abaixo).
```
if(x >= 2){
    console.log('X é ' + x);
} 
else {
    console.log("X não é menor nem igual a 2");
}
```

### else if
O else if, que significa "senão se", é utilizado após o if. Ele só será atendido caso o if não seja atendido.
```
if(x >= 2){
    console.log('X é ' + x);
} 
else if(x == 1){
    console.log("X é 1");
}
else {
    console.log("X é menor que 1");
}
```

Se a condição do primeiro if for atendida, programa pula o "else if" e o "else".
> Nós podemos usar quantos else if quisermos, assim como um if. Mas, diferente do else if, quando há várias sequências de if, todas elas serão executadas. Ou seja, devemos saber diferenciar bem else if e if. O primeiro (else if) só virá após o if e, caso a condição do if seja satisfeita, o else if não será executado. Já o segundo (if), caso haja uma sequência dele, todos serão executados mesmo que o primeiro (ou anterior) tenha a sua condição atendida.

### Switch, case e default
O condicional switch case é muito utilizado no lugar do if para lidar com várias condições.
```
switch(x){
    case 1:
        console.log(...);
        break;
    case 'impar':
        ...
        break;
    default: 
        ...        
}
```

No switch colocamos a variável que queremos verificar. Em seguida colocamos os possíveis casos e o que fazer quando esse caso for atendido. É como um if(x == y). Caso nenhum dos casos sejam atendidos, temos o default, que será executado caso nenhum dos casos sejam aceitos. Ele é colocado, geralmente, no final.

> Verifiquem que eu coloquei um "break" nos cases. Esse break serve para sair do switch para que nenhum dos outros casos sejam analizados.

Nós podemos fazer dessa forma, caso necessário:
```
switch(x){
    case 1:
    case 3:
    case 5:
    case 7:
    case 9:
        console.log("Impar");
        break;
    default:
        console.log("Par")
}
```

Esse exemplo acima poderia ser mais simples usando o operador módulo (%) em um if e else.

---

## Laços de repetição.

### for
O laço de repetição for é utilizado para fazer várias repetições até que, por fim, seja atingida a condição final. Geralmente é muito utilizado para percorrer arrays (Será falado em outra aula).

Ela funciona assim:
```
for(variável = valor ; condição; incremento ou decremento).
```

Vamos ao exemplo:
```
for(let i = 0; i <= 5; i++){
    ...
}
```
Nesse caso o laço irá se repetir até que i seja maior do que 5.

A variável pode ser declarada fora.
```
let i;

for(i = 20; i >= 10 ; i--){
    console.log("i é " + i);
}
```

### while e do while
O laço de repetição while é semelhante ao for, mas nós temos mais liberdade para montar a verificação de quando parar.
```
let x = 1000;

while(x != 0){
    x--
}

console.log("O laço terminou");
```

Nesse caso a condição é verificada primeiro. Já com o do while é diferente. A condição é vista no final.
```
let y = 1000;

do{
    y = y + 2
} while(y != 0);

console.log("O laço terminou");
```

> Nesse último código eu coloquei o "y = y + 2" para demonstrar que com while nós podemos fazer o incremento como queremos. Além disso, nós podemos por strings ou booleans para serem analizados. A liberdade é maior.