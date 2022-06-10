# Aula 4

As funções nos permitem "armazenar" um conjunto de código para ser executado quando nós chamamos ele.

## Formas
A sua sintaxe é bem simples. Há algumas maneiras de declará-las.

### Padrão
```
function nome() {
    let a = 1;
    let b = 2;
    total = a + b;
    return total;
}

nome() // Irá retornar a variável total
```

### Anônimo
Geralmente elas são armazenadas.
```
const soma = function () {
    let a = 1;
    let b = 2;
    total = a + b;
    return total;
}

soma()
```

Mas podem aparecer sozinhas e executarem a si mesma dessa forma:
```
(function() {
    let a = 1;
    let b = 2;
    total = a + b;
    return total;
} ())
```

### Arrow function
É uma forma bem mais simples de escrever uma função. Elas devem ser armazenadas em alguma variável caso ela não seja parâmetro de uma função. Nós vamos aprender sobre parâmetro mais a frente. Este será apenas um exemplo.
```
const a = () => {...}

ou

function b(p){
  return p
}
b(()=>{return 5})() 
```

Além disso, ela pode ser escrita mais simples ainda caso seja apenas 1 linha.
```
() => 5

ou

(() => 5)() 
```

---

## Parâmetros
É através dos parâmetros que nós podemos indicar os valores que a função irá analisar. Elas são definidas dentro dos parênteses. Eles são separados por vírgula.
```
function soma(a,b){
    return (a+b)
}

let n = 1
let m = 2

soma(n, m)
```

Nós podemos escrever os valores direto, caso desejarmos.
```
soma(1, 2)

ou

soma((() => 1)(), 2)
```