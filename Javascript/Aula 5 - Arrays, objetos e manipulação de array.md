# Aula 5

## Arrays
Os arrays, no javascript, servem para armazenar valores. Ao invés de criar várias variáveis para armazenar vários valores, nós podemos colocá-los em um array.
```
let a = [1, 2, 3, 4, 5];
let b = ['joão', 'ana', 'gabriel'];
```

Esse tipo de array é um array simples, mas podemos ter mais dimensões. Por exemplo:
```
let a = [[1,2,3], [4,5,6], [7,8,9]];
```

Para acessar esse array, nós devemos pegar o seu índice (que é a posição que ele está). A contagem dos índice sempre inicia a partir do 0.
Usando o exemplo acima nós temos:
```
console.log(a[0]) // Irá retornar [1, 2, 3]

console.log(a[0][1]) // Irá retornar 2
```

Podemos, também, armazenar funções:
```
let a = [function(){console.log("Oi")}] 

console.log(a[0]()); // Oi
```

## Objetos
Os objetos são variáveis que possuem "características". 
```
let pessoa = {
    id: 1,
    nome: 'Wallinson',
    programador: true
}
```

Observem como ele é feito. A atribuição de valor não é feito com "=" mas com ":". E, para separar os atributos, nós usamos vírgula.

Para pegar algum atributo, fazemos dessa forma:
```
console.log(pessoa.id) // Irá retornar 1
```

Além disso, podemos colocar funções em objetos e executá-los.
```
let dog = {
    nome: 'ralf',
    latir: function() {
        console.log("Au au")
    }
}

console.log(dog.latir()); // Au au
```

Nós ainda podemos usar em parâmetros de funções.
```
function pegarNomeCompleto({nome, sobrenome}){
	return `${nome} ${sobrenome}`;
}

let pessoa = {
    nome: 'Wallinson',
    sobrenome: 'Rocha'
}

console.log(pegarNomeCompleto(pessoa));
```

## Exploração em Objeto e Array

**Key**: Extrai as palavras chaves de um objeto/array:
```
Object.keys(objeto ou array);
```

**Values**: Extrai os resultados das palavras chaves:
```
Object.values(objeto ou array);
```

**Entries**: Retorna o id, chave e valor:
```
Object.entries(objeto ou array);
```

## Manipulação de arrays

### array.toString() - Transformar array em string
```
let lista = ['ovo', 'pão', 'arroz'];
let n = lista.toString();
console.log(n);
// ovo,pão,arroz
```
Ele junta todos os valores e separa-os com vírgula.

### array.join() - Transformar array em string
Com o método join, nós escolhemos no parâmetro como ele deve ser separado, se por espaço, virgula, ponto e vírgula ou outro:
```
let lista = ['ovo', 'pão', 'arroz'];
let n = lista.join(', ');
// ovo, pão, arroz
```

### array.indexOf() - Procurar letra ou palavra na lista
Da mesma forma que é usado na manipulação de string (próxima aula), podemos usar o indexOf para achar valores
```
let n = lista.indexOf('ovo');
```

### array.pop() - Tirar item
Pop( )
```
let lista = ['ovo', 'arroz', 'pão'];
lista.pop();
```
Caso não seja especificado nada no parâmetro, ele irá remover o último.

### array.push() Adicionar item
Push( )
```
let lista = [];
lista.push('ovo');
console.log(lista)
// ovo
```

### delete array - Limpar array
Com o delete nós podemos limpar uma array:
```
let lista = ['1', '2', '3'];
delete lista;
// []
```

### array.splice() - Excluir série de itens 
A função .splice() pode deletar uma série de itens quando especificado em seus parâmetros.
```
let lista = ['1', '2', '3'];
lista.splice(1, 2);
// lista == ['1']
```
Neste caso foi especificado que a partir da posição 1 seriam deletados 2 números. Caso não seja especificado o número de itens a ser excluído, a partir do ponto de partida todos os itens serão excluídos.

### array.splice() - Adicionar ou substituir
Além disso, podemos adicionar algum elemento em alguma posição. Para isso devermos passar 3 (mínimo) parâmetros: posição, 0, elemento1, elemento2...
```
let lista = ['1', '2', '3'];
lista.splice(1, 0, '1.5');
// lista == ['1', '1,5', '2', '3']
```

Caso queira substituir, no ludar do 0, colocamos 1.

### array.concact - Juntar arrays
.concat(lista)
```
let lista1 = ['ovo', 'carne'];
let lista2 = ['água', 'laranja'];

lista1.concat(lista2)

// ['ovo', 'carne', 'água', 'laranja']
```
Com isso, as duas listas estarão juntas numa só, no caso a anterior.

Esse método não é tão utilizado, mas com o operador spread:
```
let a = [1, 2, 3]
let b = [...a, 4, 5, 6]
```

### array.sort() - Ordenação crescente
```
lista.sort()
```
Com essa função, a lista será organizada em forma alfabética ou de forma crescente numericamente.

### array.reverse() - Ordenação decrescente
```
lista.reverse()
```
Com essa função, a lista será organizada de forma decrescente seja numérica ou alfabética.

### array.map() - Mapeação de Array
Com a função map(), o array que recebeu essa função será mapeada e será executada uma função para cada elemento:
```
let lista = [1, 2, 3];
let lista2 = [];

lista2 = lista.map(function(item){
    return item * 2;
})
```
Nesse caso, a lista2 herda os valores da lista1 porém duplicada.

### array.filter - Filtro de array
Com a função filter(), podemos filtrar o array e admitir alguns valores.
```
let lista = [1, 2, 3];
let lista2 = [];

lista2 = lista.filter(function(item){
    if(item < 20){
        return true;
    } else{
        return false
    }
});
```
Nesse caso, todos os valores menores do que 20 serão admitidos.

### array.every() - Validação se todos
Com a função every(), todo os elementos devem atender os requisitos da condição. Caso contrário o retorno será falso.
```
let lista = [1, 2, 3];
let lista2 = [];

lista2 = lista.every(function(item){
    if(item < 20){
        return true;
    } else{
        return false
    }
});
```
Nesse caso retornaria true pois TODOS os itens são menores do que 20. Caso tivesse algum maior ou igual a 20, a função retornaria false.

### array.some() - Validação se 1
Com a função some(), deverá haver apenas um elemento que atenda os requisitos da condição para que retorne true.
```
let lista = [1, 2, 3, 50, 79];
let lista2 = [];

lista2 = lista.every(function(item){
    if(item < 20){
        return true;
    } else{
        return false
    }
});
```
Nesse caso retornaria true pois ALGUNS itens são menores do que 20. Caso não houvesse nenhum, retornaria false.

### array.find() - Verificar existência
Com a função find() nós podemos procurar alguns resultados em alguma lista ou objeto, por exemplo.
```
let lista = [
    {id: 1, nome: 'Bonieky', sobrenome: 'Lacerda'},
    {id: 2, nome: 'Paulo, sobrenome: 'XYZ'},
    {id: 3, nome: 'Fulano', sobrenome: 'Da Silva'}
];

let pessoa = lista.find(function(item){
    return (item.id == 3) ? true : false;
});
```
Nesse caso ele retornará os dados que possui o id == 3.