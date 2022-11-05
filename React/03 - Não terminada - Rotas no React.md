# Routes

## Instalação

```
yarn add react-router-dom
```

## Importação

```
import {BrowserRouter as Router, Routes, Route, Link} from 'react-router-dom';
```

## Estrutura

```
<Router>
    <ul>
        <li>
        <Link to="/">Home</Link>
        </li>
        <li>
        <Link to="/sobre">Sobre</Link>
        </li>
        <li>
        <Link to="/produtos/esportes">Produtos - esportes</Link>
        </li>
    </ul>
    <Routes>
        <Route path="/" element={<Home/>}/>
        <Route path="/sobre" element={<Sobre/>}/>
        <Route path="/produtos/:cat" element={<Produtos/>}/>
    </Routes>    
</Router>
```

O **Router** envolve todas as rotas. 
**Link to** faz o direcionamento para as routas.
**Routes** faz a troca para atender as rotas.
**Route path** element realiza a troca para mostrar aquilo que foi direcionado.

Caso ele seja integrado de forma separada, o **Router** deve englobar toda a aplicação.

```
<Router basename={process.env.PUBLIC_URL}>
    <Header /> // Aqui dentro há o Link
    <AreaToDo/> // Aqui dentro há o Routes com as Route
    <footer>Desenvolvido por Wallinson Rocha.</footer>
</Router>
```

## useParams e useNavigate

### Importação

```
import { useParams, useNavigate } from "react-router"
```

### Estrutura

```
export default function Produtos(){
    
    const { cat } = useParams()
    const navigate = useNavigate();

    return(
        <div>
            <p>
                Produtos {cat}
            </p>
            <button onClick={() => navigate("/")}>Home</button>
        </div>
    )
}
```

Para que o **useParams** seja usado, podemos especificar na nossa url o valor que ele deve receber. O seu valor é captado em onde houver a sua referência, será substituído.

```
<Link to="/produtos/esportes">Produtos - esportes</Link>
<Routes>
    <Route path="/produtos/:cat" element={<Produtos/>}/>
</Routes>
```

Com o useParams, nós podemos fazer o direcionamento a alguma categoria que nós especificamos na rota. É muito comum ser usado com as rendesizações.

```
function Invoice() {
  let { invoiceId } = useParams();
  let invoice = useFakeFetch(`/api/invoices/${invoiceId}`);
  return invoice ? (
    <div>
      <h1>{invoice.customerName}</h1>
    </div>
  ) : (
    <Loading />
  );
}
```

Por sua vez, o **useNavigate** serve para fazer direcionamentos diretos como especificado no código abaixo que foi retirado do código mais acima.

```
export default function Produtos(){
    const { cat } = useParams()
    const navigate = useNavigate(); // Observe aqui
    return(
        <div>
            <p>
                Produtos {cat}
            </p>
            <button onClick={() => navigate("/")}>Home</button> // Observe aqui
        </div>
    )
}
```

## useLocation

### Importação

```
import { useLocation } from 'react-router';
```

### Utilização

Retorna o objeto de localização que representa a URL atual. Você pode pensar nisso como um useState que retorna um novo local sempre que a URL muda.
Isso pode ser realmente útil, por exemplo: em uma situação em que você deseja acionar um novo evento de “visualização de página” usando sua ferramenta de análise da web sempre que uma nova página é carregada, como no exemplo a seguir:

```
const [name, setName] = useState("");
    let location = useLocation()
    useEffect(() => {
        switch(location.pathname){
            case "/": setName("Todos"); break;
            case "/sem-data-e-hora": setName("Sem data e hora"); break;
            case "/hoje": setName("Hoje"); break;
            case "/esta-semana": setName("Esta semana"); break;
            case "/este-mes": setName("Este mês"); break;
            case "/este-ano": setName("Este ano"); break;
        }
    },[location.pathname])
```