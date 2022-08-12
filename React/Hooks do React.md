# Hooks do React

São hooks que vem com o próprio React.

## useState - Importação

```
import React, {useState} from 'react';
```

## useState - Utilização

```
function App(){
    const [ texto, setTexto ] = useState('');
```

## useEffect - Importação

```
import {useEffect} from 'react';
```

## useEffect - Utilização

Ele é, geralmente, usado com useState, mas pode ser usado de forma independente.

```
const [contagem, setContagem] = useState(0);

useEffect(() => {
	if(contagem == 0){
		document.title = "Começou a brincadeira!";
	} else{
		document.title = "Contagem: "+contagem;
	}
}, [contagem]);

function aumentarAction() {
	setContagem(contagem+1);
}
```

Se quisermos que ele rode ao inicializar a aplicação, basta deixarmos o verificador vazio.

```
useEffect(() => {
	...
}, []);
```