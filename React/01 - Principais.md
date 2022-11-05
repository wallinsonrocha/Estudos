# Principais 

## Criação

Nesse caso estou usando o Yarn. Para iniciar um projeto, iniciarei ele através do Vite.

```
yarn create vite my-app --template react
```

Após isso, precisamos instalar as principais dependências do Yarn.

```
yarn
```

Para que a aplicação seja iniciada, basta rodarmos esse comando na pasta.

```
yarn dev
```

## Criação de componentes

### Classe

Podemos criar através de classes.

```
import { Component } from "react";

export default class Main extends Componentes{
	render(){
		return <h1>Teste</h1>
	}
}
```

### Funções

Podemos criar através de funções

```
export default function App(){
	return <h1>Teste</h1>
}
```

## Props

Através das props podemos enviar valores para serem processados na criação do componente.

```
export defualt function Ola(props){
	return <h1>Bem vindo, {props.nome}</h1>;
}
```

```
function App(){
	return (
		<Ola nome="Amanda"/>
		<Ola nome="Aline"/>
		<Ola nome="Raissa"/>		
	)            
}
```

Podemos usar os props para acessar **objetos**.

```
export default function Avatar(props){
	return(
		<div>
			<img src={props.user.url} alt={props.user.name}/>
			<br/>
			<span>{props.user.name}</span>
		</div>
	);
}
```

```
function App(){
	
	let user = {
		url:"http...",
		name:"Wallinson"
	};

	return(
		<Avatar user={user}/>
	)
}
```

Podemos passar alguma funcionalidade do useState para ser enviada e alter um componente.

```
export default function ManuArea(props){

	const[content, setContent] = useState('Todos');

	useEffect(()=>{
		props.setName(content);
	}, [content]);

	return(
		<div>
			<img src={props.user.url} alt={props.user.name}/>
			<br/>
			<span>{props.user.name}</span>
		</div>
	);
}
```

```
function App(){
	return(
		<MenuArea setName={setName}/>
	)
}
```

## Condicionais de exibição

```
function App(){     

    const [email, setEmal] = useState("");             

    return(   
        <>              
            <Input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
            {email.length > 0 && (
                <>                          
                	<p>{email.length} caractere{email.length == 1  ? '' : 's'}</p>     
                    <p>Outro conteúdo</p>    
                </>  
            )}       
        </> 
        )
}
```