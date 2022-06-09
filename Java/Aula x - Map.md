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