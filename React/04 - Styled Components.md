# Styled Components

## Instalação e Importação

```
yarn add styled-components --save

ou

npm install styled-components
```

```
import styled from 'styled-components';
```

## Criação 

Preferencialmente em um arquivo separado.

```
export const Site = styled.h1`
    width: 40px;
    background-color: #FF0000;
`

export const Botao = styled.button`
    font-size: 19px;
    padding: 15px;
`
```

## Aplicação

No componente desejado.

```
function App(){
    return(
        <Site>
            <Botao>Clique aqui</Botao>
        </Site>
    );
}

export default App;
```

## Javascript no CSS

Podemos colocar uma props em nosso componente para que o seu comportamento possa ser ajustado pelo JS.


Este primeiro configura uma cor padrão caso não haja alguma cor especificada na props.

```
const Botao = style.button`
	font-size: 19px;
	padding: 10px 15px;
	background-color: ${props => props.color || '#00FF00'};
`;

<Botao color="#FF0000">Clique aqui</Botao>
```

Por sua vez, este abaixo é alterado com condições.

```
const Botao2 = style.button`
	font-size: 19px;
	padding: 10px 15px;
	background-color: ${props => props.ativo == true ? '#0000FF' : '#CCC'};
`;

<Botao2 ativo={true}>Clique aqui</Botao2>
<Botao2 ativo={false}>Clique aqui</Botao2>
```

## Estendendo características

Nesse caso, o BotaoPequeno recebe todos os atributos de Boton. Além disso, pode substituir alguns atributos modificando-os.

```
const Buton = style.button`
	font-size: 19px;
	padding: 10px 15px;
	border: 3px dashed #FF0000;
	color: #FF0000;
	background-color: #FFF;
	margin: 5px;
	border-radius:5px;
`;

const BotaoPequeno = style(Buton)`
	padding: 5px 10px;
	font-size: 16px;
`;
```

## Aplicando em input[type]

Dessa forma o componente Input já tem o seu tipo especificado.

```
input.attrs({ type: 'submit' })`
    ...
`
```

## Modificando descendentes

A sua escrita também é bem semelhante ao SCSS.

```
const Test = style.div`
	font-size: 19px;
	padding: 10px 15px;
	border: 3px dashed #FF0000;
	color: #FF0000;
	background-color: #FFF;
	margin: 5px;
	border-radius:5px;

        img{
            //CSS
        }

        button{
            //CSS
        }

        &:hover:before{
            //CSS
        }
`;
```

Essa indentação que fiz é uma preferência minha para melhorar a leitura do código.