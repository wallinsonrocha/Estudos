Como nós vimos na aula passada, podemos definir qual será o tipo que alguma variável terá.

## Tipos básicos
Os tipos mais usados são:
### string
Com ela nós declaramos que a variável é do tipo string.
```
let nome: string = 'Wallinson';
```
### number
Definimos que ela é um número
```
let idade: number = 21;
```
### boolean
Definimos se ela terá o valor de verdadeiro ou falso
```
let adulto: boolean = true;
```

## É obrigatório colocar o tipo quando seguido de algum valor?
Não, não é obrigatório. Se nós declaramos uma variável e, em seguida, depositamos algum valor nela, o seu tipo será definido automaticamente.
```
let nome = 'Wallinson' // Esse tipo será, automaticamente, string
```

## Com array
Para trabalhar com os arrays, na declaração dela, devemos atribuir o seu tipo dessa forma:
### Array<tipo>
```
let numberArray: Array<number> = [1, 2, 3, 4, 5];
```
```
let stringArray: string[] = ['1', '2', '3', '4', '5'];
```

Caso seja necessário, nós podemos atribuir o "readonly" para que ela seja imutável.
```
const numeros: readyonly number[] = [1, 2];
const nomes: ReadyonlyArray<string> = ['Olivia', 'Maria'];
```

## Com objetos
Com objetos o sistema é semelhante
```
let pessoa: {nome: string; idade: number; adulto?: boolean} = {
    nome: 'Sarah',
    idade: 21,
    adulto: true
};
```
> O '?' indica dizer que não somos obrigado a especificá-lo. Caso não fosse especificado, ele será undefinid.

## Em funções
Semelhante ao objeto
```
function Soma(x: number, y:number){
    return x+y;
}
```
> Nesse caso, ele irá retornar um number.

## Tipo any e void
### Any
O tipo any é o que, geralmente, não vamos querer em nosso código. Ele não retorna um tipo definido. Geralmente pode ser usado em funções. Esse tipo, se declarado propositalmente, é usado em último caso ou em casos muito específico onde o retorno de uma função pode ser de qualquer tipo.
```
function showMensage(msg){
    return msg
}

console.log(showMensage([1, 2, 3]));
console.log(showMensage('Olá'));
console.log(showMensage(1));
```
### Void
O tipo void, geramente utilizada em funções ou métodos, é usada quando não há retorno algum.
```
function noReturn(...args: string[]): void{
    console.log(args.join(' '));
}

noReturn('Wallinson', 'Sarah', 'Nicolas', 'Sophia', 'Nicole');
```
> Nesse caso, o operador rest irá ler um array do tipo string. Nesse caso a sua especificação para tipagem é "string[]".

## Tipo tupla
O tipo tupla não é nativo do javascript, mas podemos ter acesso a ele com o typescript. Mas, diferente da linguagem python, nós podemo mudar os seu valores.
```
const dadosCliente1: [number, string] = [1, 'Luiz'];

dadosCliente1[0] = 100;
dadosCliente2[1] = 'Camelo'
```

> É um array com tipos.

Caso seja necessário acrecentar mais strings dentro da tupla, nós podemos resolver esse problema com o operador rest
```
const dadosCliente3: [number, string, ...string[]] = [1, 'Luiz']
```
Nesse caso eu estaria especificando que no primeiro parâmetro deveria ter uma chave de valor number, o segundo como string e os demais como string também.

Caso seja necessário, nós podemos atribuir à tupla o "readonly" para que ela seja imutável.
> Isso pode ser feito em objetos e arrays.
```
const dadosCliente3: readonly [number, string, ...string[]] = [1, 'Luiz']
```

## Union Type e Intersection Type
### Union type
Serve para indicar que um parâmetro ou variável pode admitir um ou outro tipo.
```
function AddOrConcat(a: number | string, b: number | string): number | string {
    if (typeof a === 'number' && typeof b === 'number') return a+b;
    return `${a} e ${b}`;
}

console.log(10, 20); // 30
console.log (10, '20') // 10 e 20
```

### Intersection type
Serve para indicar que um parâmetro ou variável deve ter um tipo ou outro tipo.
```
type Nome = {nome: string};
type Sobrenome = {sobrenome: string};
type Idade = {idade: number};

type Pessoa = Nome & Sobrenome & Idade;
```
Dessa maneira, o tipo Pessoa é obrigado a ter todos os tipos que foram impostas a ela.

## Type Alias
É um tipo personalizado por nós. Geralmente é utilizado para tipar objetos.
```
type Idade = number;
type Pessoa = {
    nome: string;
    idade: Idade;
    salario: number;
    corPreferida?: string;
} 

const pessoa: Pessoa = {
    nome: 'Cleiton';
    idade: 30;
    salario: 8000;
}
```

Podemos até mesmo combinar com tipos literais e Union type:
```
type CorRGB = 'red' | 'green' | 'blue';
type CorCMYB = 'Cian' | 'Magenta' | 'Amarelo' | 'Preto';
type CorPreferida = CorRGB | CorCMYB;

function setCorPreferida(pessoa: Pessoa, cor: corPreferida): Pessoa {
    return(...pessoa, corPreferida: cor);
}
```

## Tipo com função
Aqui nós podemos criar um tipo para ser aplicada em funções. Dessa forma nós podemos retornar o tipo do valor:
```
type User = {username: string; password: string};
type VerifyUserFn = (user: User, sentValue: User) => boolean;

const verifyUser: VerifyUserFn = (user, sentValue) => {
    return(
        user.username === sentValue.username && user.password === sentValue.password
    );
}

const bdUser = {username: 'joao', password: '123456'};
const sentUser = {username: 'joao', password: '123456'};

const loggedIn = verifyUser(bdUser, sentUser); 
console.log(loggedIn) //true
``` 