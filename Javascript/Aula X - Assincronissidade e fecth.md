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
Usando método POST
```
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

Usando método GET
```
fecth('https://endereco-api.com/', {
    method: 'GET',
    cache: 'no-cache'
})
    .then(response => response.json())
    .then(json => console.log(json))

// Retorna uma promise
```

## Alguns métodos
1. POST - Criar
2. GET - Pegar
3. PUT - Atualizar
4. DELETE - Deletar