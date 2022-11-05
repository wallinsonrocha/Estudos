# Redux

## Instalação
```
npm install @reduxjs/toolkit react-redux
```

## INÍCIO
Devemos criar um arquivo chamado store.js. Lá iremos importar o configureStore :
```
import { configureStore } from '@reduxjs/toolkit'

export default configureStore({
  reducer: {}
})
```
O configureStore leva um objeto chamado reducer que retorna mais objetos.

É através dela que algumas informações serão mantidas.
Além disso, devemos enviar alguma função dentro do createStore. Nesse caso, devemos enviar alguns reducers. Isso será abordado em Criação do Reducer.

E, para integrá-la, devemos pô-la no Provider onde receberá uma propriedade chamada store. Lá iremos colocar a constante que armazenamos o createStore. No nosso caso, store. 

### No index.js:
```
import store from './store'
import {Provider} from 'react-redux'

<Provider store={store}>
    <App/>
</Provider>
```

Ao criar a pasta, devemos criar o arquivo que será o nosso reducer. Ele poderá levar qualquer nome. Nesse caso, vamos supor que seja criado um reducer com o nome usuarioReducer:
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