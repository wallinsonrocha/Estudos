# Redux - Persist

## Instalação
```
npm install redux-persist
```

## Store.js
No arquivo **store.js**, devemos configurar assim:
```
import { configureStore, combineReducers } from "@reduxjs/toolkit";
import listReducer from "./features/listSlice";
import { persistReducer, persistStore } from "redux-persist";
import storage from "redux-persist/lib/storage";

const rootReducer = combineReducers({
    todos: listReducer,
});

const persistConfig = {
    key:'root',
    storage
}

const pReducer = persistReducer(persistConfig, rootReducer);

const store = configureStore({
    reducer: pReducer
})

export const persistor = persistStore(store);
export default store;
```

## Index.js
Após esses procedimentos, no **index.js** do App:
```
import {PersistGate} from 'redux-persist/integration/react';
import {store, persistor} from './store';
...
<PersistGate loading={null} persistor={persistor}>
    <App/>
</PersistGate>
```

Recomenda-se criar o store dessa forma caso seja necessário persistir os dados.

Para mais informações, consultar a [documentação](https://www.npmjs.com/package/redux-persist)