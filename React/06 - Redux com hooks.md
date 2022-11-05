# Redux - Hooks 

Os Hooks para o Redux simplificam e muito o processo de connect e de dispatch. Através desses hooks podemos simplificar esse processo em poucas linhas.
Nós a usamos nos componentes que queremos ter acesso.

## Importação
```
import {useSelector, useDispatch} from 'react-redux';
import {setName} from './usuarioReducer';
```
Importamos tanto os Hooks como também os actions.

## Armazenando em alguma variável ou constante:
### Connect

```
const name = useSelector(state => state.user.name);
```

### Dispatch

```
const dispatch = useDispatch()

const handleButton = () => {
    dispatch(setName('Felipe'));
}
```

Para utilizar a variável, não precisamos configura-la como uma props. Basta por o nome da variável.
```
...
<p>Olá, {name}</p>
...
```