# Aula x

O Set, as coisas são um pouco diferente. Diferente do ArrayList e do Map, o Set não contém índices e, como o List, não contém chave e valor.

## Criação de um Set 
```
Set<Tipo> set = HashSet<>(Array.asList(valor1, valor2, valor3, valor4...));

ou

Set<Tipo> set = TreeSet<>(objeto);
```

## Implementação
Existem 3 implementações para o set.
HashSet - Não ordenado
TreeSet - Ordenado
LinkedHashSet - Mantém a ordem que os elementos foram incluídos.

## Alguns métodos
### Contains
```
set.contains(object);
```
Verifica se existe ou não determinado elemento retornando true ou false.

### Size
```
set.size();
```
Exibe o tamanho da set.

### Remove
```
set.remove(object)
```
Serve para remover algum objeto no set. Para isso, basta escrever o que será removido.

### Add
```
set.add(e);
```
Através dele nós podemos acidionar elementos em nosso set. Diferente da lista, ele só têm como argumento o elemento(e).

### Clear
```
set.clear();
```
Irá limpar todo o set.

### IsEmpty
```
set.isEmpty();
```
Nesse caso, ele irá apenas fazer a verificação se o set está vazio ou não.

---

## Métodos com Collections

### Min
```
Collections.min(set);
```
Levando em consideração uma set do tipo double, int ou qualquer outra forma numeral, com isso irá retornar o menor valor do set selecionada.

### Max
```
Collections.max(set);
```
Levando em consideração um set do tipo double, int ou qualquer outra forma numeral, com isso irá retornar o maior valor do set selecionada.

### Iterator
O iterator serve para verificar, em um laço de repetição, se há algum próximo elemento. Ele pode ser utilizado para fazer a soma em um set de números. Ele deve ser importado.
Exemplo:
```
Iterator<Double> iterator = notas.iterator(); //set notas
Double soma = 0d;

while(iterator.hasNext()){
    Double next = iterator.next();
    soma += next;
}
```
No caso dessa lógica, irá verificar se há mais elementos no set. Caso tenha, a variável next irá receber o próximo valor através do iterator.next();

### Shuffle
Irá dispor os elementos de forma aleatória.
```
Collections.shuffle(set);
```

### Sort
Supomos que criamos uma classe gato que receberá vários valores sobre os gatos que estarão no set. Para organizar em ordem alfabética, nós devemos implementar a interface Comparable nela e adicionar o compareTo. Somente após isso, na classe main que nós poderemos organizar.
```
class Gato implementes Compareble<Gato>{
    ...
    @Override
    public int compareTo(Gato gato){
        return this.getNome().compareToIgnoreCase.(gato.getNome());
    }
}
```

Caso seja necessário comparar as idade, basta criar outra classe:
```
class ComparatorIdade implementes Comparator<Gato>{
    ...
    @Override
    public int compareTo(Gato g1, Gato g2){
        return Integer.compare(g1.getIdade(), g2.getIdades);
    }
}
```

Logo após isso, na classe Main, podemos por:
```
Collections.sort(gato);

ou, para ideade

Collections.sort(gato, new ComparatorIdade());
```