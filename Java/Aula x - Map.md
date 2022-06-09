# Aula X

Com o Map nós podemos fazer uma organização de chave e valor. Para cada chave nós temos um valor. A chave não poderá ser repetida.



## Declaração
```
Map<Tipo1, tipo2> nome = HashMap<>(){{
    put('chave', valor);
}};
```

## Implementação
Existem 3 implementações para o map.
HashMap - Não ordenado
TreeMap - Ordenado
LinkedHashMap - Mantém a ordem que os elementos foram incluídos.

### Alguns métodos
#### Put
Com o método put nós podemos adicionar ou substituir alguma chave no Map ou no Set.
```
Map<Tipo1, tipo2> nome = HashMap<>(){{
    put('chave', valor);
}};

ou, caso já formado

nome.put('chave', valor);
```

#### ContainsKey
Para verificar se existe tal chave.
```
nome.containsKey('chave');
```

#### Get
Para checar o valor da chave.
```
nome.get('chave');
```

#### KeySet
Para exibir as chaves do map retornando um Set.
```
Set<tipo da chave> modelos = carros.keySet();
```

#### Values
Para exibir os valores das chaves retornando um Collections.
```
Collections<tipo do valor> colection = nome.values();
```









Iterator<String> contador = respostas.iterator();
        while(contador.hasNext()){
            String resp = contador.next();
            if(resp.contains("s")) {
                count ++;
            }
        }




Lambda
numerosAleatorios.stream().forEach(s -> System.out.print(s));

Com a operação lambda podemos simplificar o código. Mas ele pode ser simplificado mais ainda como um metodo de referencia. Assim:

numerosAleatorios.stream().forEach(System.out::println);

É como se a referência fosse os dois pontos. Um exemplo é com o map:

numeros.stream().map(Integer::parseInt);

ao invés de

map(s -> Integer.parseInt(s));




Método Stream
numeros.stream()
    .limit(5)
    .collect(Collectors.toSet())
    .forEach(System.out::println);

Set<String> nm = numeros
    .limit(5)
    .collect(Collectors.toSet());





Stack (pilha)

Stack<object> nome = new Stack<>();

Métodos:
push() - adicionar
pop() - remover o último
peek() - Verifica o último item da pilha.
search() -
empty() - Verifica se está vazia.