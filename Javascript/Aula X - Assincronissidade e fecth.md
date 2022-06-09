# Aula X
Na asincronissidade do Javascript, podemos orientar alguma função de maneira assincrona. Ou seja, podemos executar duas coisas ao mesmo tempo.

## Promise
A promise, como diz o nome, é literalmente uma promessa. Quando executamos alguma função e damos a promise, ele irá ser executar assim que um dos os seus parâmetros forem atendidos.
```
const myPromisse = new Promise((resolve, reject) => {
    window.setTimeout(()=>{
        resolve(console.log('Resolvida'));
    }, 2000)
});
```

ou, em uma função:
```
function fn()=>{
    return myPromisse = new Promise((resolve, reject) => {
        window.setTimeout(()=>{
            resolve(console.log('Resolvida'));
        }, 2000)
    });
}
```

No exemplo acima, após 2s a mensagem irá aparecer. Mas, enquanto isso, caso haja mais códigos depois da promise, eles serão executados. Ou seja, o programa não irá esperar 2 segundos para depois seguir. Ela irá ser executada de forma Assincrona.

Nós podemos pegar o seu valor e usar no .then para outras funções. Mas, caso haja algum erro, podemos usar o catch para capturar esse erro.
```
const resolved = await myPromisse
    .then((result) => result + 'passando pelo then')
    .then((result) => result + e 'agora acabou!');
    .catch((err) => console.log(err.mensage));
```

## Async e await
Essas duas palavras são usadas para trabalhar com a promisse (principalmente o await).
O async serve para definir uma função como assincrona. Ela se faz necessária para que o código identifique-a como assincrona e seja reconhecida no await. E, neste, o código reconhcerá que tem uma função assincrona que deverá se terminada antes de prosseguir. Ela é ideil para lidar com Promises.
```
async function resolvePromise(){
    const myPromise = new Promise((resolve, reject) => {
        window.setTimeout(()=>{
            resolve(console.log('Resolvida'));
        }, 3000);
    });

    const resolved = await myPromisse
        .then((result) => result + 'passando pelo then')
        .then((result) => result + e 'agora acabou!');
        .catch((err) => console.log(err.mensage));
    
    return resolved;
}
```

Pode ser escrita, também:
```
const asincFn = async () => {}
```

## fecth
O fecth é usado para coletar APIs. Para isso, é necessário coletar a url/arquivo que disponibiliza essa API. A sua sintaxe é:
```
fecth(url, options)
    .then(response => response.json())
    .then(json => console.log(json))

// Retorna uma promise
```

Ela, dentro de uma função async, pode ser acompanhada com o await.
```
async function fecthMovies(){
    const response = await fecth('/movies');
    return await response.json();
}
```

Exemplo:
### Usando método POST
```
const data = ...
fecth('https://endereco-api.com/', {
    method: 'POST',
    cache: 'no-cache',
    body: JSON.stringfy(data)
})
    .then(response => response.json())
    .then(json => console.log(json))

// Retorna uma promise
```
Com os métodos POST nós enviamos algo para algum banco de dado ou algo do tipo. Quando usamos o POST, devemos converter para Json usando JSON.strinfy().

### Usando método GET
```
fecth('https://endereco-api.com/')
    .then(response => response.json())
    .then(json => console.log(json))

// Retorna uma promise
```

### Usando o método Delete
```
fecth('https://endereco-api.com/cliente/4', {
    mothod: 'delete'
})
    .then(response => response.json())
```

### Usando o método Put:
```
const data = ...
fecth('https://endereco-api.com/cliente/4', {
    mothod: 'put',
    body: JSON.stringify(data)
})
    .then(response => response.json())
```

## Alguns métodos
1. POST - Criar
2. GET - Pegar
3. PUT - Atualizar
4. DELETE - Deletar

# Axios
O axios é uma api que trabalha com o XmlHttpRequest (ou seja, diferente do fecth, ele consegue trabalhar em navegadores mais antgos).
Para instalar o axios, no terminal digitamos:
```
npm install axios
```
E, em seguida devemos importá-lo.
```
import axios from 'axios';
```

### Usando método POST
```
const data = ...
axios.post('https://endereco-api.com/', {
    nome: 'Nome de algúem',
    endereco: 'Rua tal casa tal',
    area: 'letras',
    mais: JSON.stringify(data) //Constante
})
    .then(response => response.json())
    .then(json => console.log(json))

// Retorna uma promise
```

### Usando método GET
```
axios.get('https://endereco-api.com/')
    .then(response => response.json())
    .then(json => console.log(json))

// Retorna uma promise
```

### Usando o método Delete
```
axios.delete('https://endereco-api.com/cliente/4')
    .then(response => response.json())
```

### Usando o método Put:
```
const data = ...
axios.put('https://endereco-api.com/cliente/4', {
    nome: 'Nome de algúem',
    endereco: 'Rua tal casa tal',
    area: 'letras',
    mais: JSON.stringify(data) //Constante
})
    .then(response => response.json())
```