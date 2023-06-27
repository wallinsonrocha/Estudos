# Redux

## Instalação
Redux
```
# NPM
npm install redux react-redux

# Yarn
yarn add redux react-redux
```
Com redux tool kit
```
# NPM
npm install @reduxjs/toolkit react-redux

# Yarn
yarn add @reduxjs/toolkit react-redux
```

## INÍCIO
Devemos criar um arquivo chamado store.js. Lá iremos importar o createStore :
```
import { createStore } from 'redux'

function counterReducer(state = { value: 0 }, action) {
  switch (action.type) {
    case 'counter/incremented':
      return { value: state.value + 1 }
    case 'counter/decremented':
      return { value: state.value - 1 }
    default:
      return state
  }
}

let store = createStore(counterReducer)

// Você pode usar o método subscribe() para atualizar a interface do usuário em resposta às alterações no estado.
// Normalmente, você usaria uma biblioteca de ligação de visualização (por exemplo, o React Redux) em vez de usar o subscribe() diretamente.
// Pode haver casos adicionais em que é útil também fazer uma inscrição (subscribe).

store.subscribe(() => console.log(store.getState()))
```

### Com o redux tool kit
Redux tool kit é configureStore.

```
import {createSlice} from '@reduxjs/toolkit'

export const usuarioReducer = createSlice({
    name: 'usuario',
    initialState:{
        name: 'Visitante'
    },
    reducers: {
        setName: (state, action) => {
            state.name = action.payload
        },
        ...
    }
})
```
O configureStore leva um objeto chamado reducer que retorna mais objetos.

É através dela que algumas informações serão mantidas.
Além disso, devemos enviar alguma função dentro do createStore. Nesse caso, devemos enviar alguns reducers. Isso será abordado em Criação do Reducer.

E, para integrá-la, devemos pô-la no Provider onde receberá uma propriedade chamada store. Lá iremos colocar a constante que armazenamos o createStore. No nosso caso, store. A mesma coisa funciona caos não esteja usando o Redux Tool Kit.

Dentro irá retornar objetos com name, initialState e reducers.

**state** é uma propriedade que armazena informações. Como o nome diz é um estado.
**action** é a ação que será feita para modificar o estado (state). Nem sempre é necessário usara, a não ser que sera necessário passar algum parâmetro.

Caso seja necessário adicionar um objeto onde cala chave deverá ter o seu valor, devemos indexar algum nome referencial após o payload. Isso serve para que, quando for passado um objeto como parâmetro, eles sejam reconhecidos dentro do objeto e retorne os seus devidos valores.
```
addTodo: (state, action) => {
            let newAtv = {
                title: action.payload.title,
                data: action.payload.data,
                hour: action.payload.hour,
                key: state.todoList.length,
                checked: false
            }
            state.todoList.push(newAtv)
        }

        ...
    
function add(){
        let newAtv = {
            title: textActivity,
            data: dataActivity,
            hour: hourActivity
        }
        dispatch(addTodo(newAtv));
    }
```
Dessa forma, conforme o exemplo acima, os valores do objeto newAtv irão ser reconhecidos ao ser usado a função dispatch. Ele irá pegar esses valores e posiciona-los de acordo com o payload de cada um.

Logo em seguida, precisamos exportar alguns arquivos:
```
export const {setName} = usuarioReducer.actions;
export default usuarioReducer.reducer;
```

### No index.js:
```
import store from './store'
import {Provider} from 'react-redux'

<Provider store={store}>
    <App/>
</Provider>
```

Ao criar a pasta, devemos criar o arquivo que será o nosso reducer. Ele poderá levar qualquer nome. Nesse caso, vamos supor que seja criado um reducer com o nome usuarioReducer:

## ADICIONANDO REDUCER
Após isso, precisamos importar o reducer criado no arquivo **store.js**
```
import usuarioReducer from './reducer/usuarioReducer'
...
  reducer: {
      user: usuarioReducer
  }
...
```
