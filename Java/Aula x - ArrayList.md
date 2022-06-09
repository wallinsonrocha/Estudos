# Aula x
No java, quando declaramos algum array e trabalhamos com ele, percebemos o quanto é difícil trabalhar com ele em alguns momentos:
- Não podemos redimensionar uma array em Java;
- É impossível buscar diretamente por um determinado elemento cujo índice não se sabe;
- Não conseguimos saber quantas posições da array já foram populadas sem criar, para isso, métodos auxiliares.

> Parte extraída do site da Alura na apostila de Java em Collections framework.

Para resolver esses problemas, o Java implementou a Collection, que é uma API que representa estruturas de dados avançados.

## List ou ArrayList
A list, que está disponível no pacote java.util.List é uma das primeiras alternativas para resolver esses problemas.
> Obs.: Apesar de ter o nome Array, a List não é uma.

## Criação de um ArrayList:
```
ArrayList lista = new ArrayList();
ou
List lista = new ArrayList();
ou
List<tipo> lista = new ArrayList<tipo>();
```

## Alguns métodos
### Add
```
lista.add(i, e);
```
Através dele nós podemos acidionar elementos em nossa lista. Ele têm como argumento a posição (i) e o elemento(e). Caso seja declarado apenas 1 argumento, ele será o elemento e a sua posição será a próxima. Ou seja, como em uma pilha, ele irá adicionar após o último.

### Remove
```
lista.remove(object)
```
Serve para remover algum objeto na lista. Para isso, basta escrever o que será removido.

### Size
```
lista.size();
```
Exibe o tamanho da lista.

### Get
```
lista.get(int);
```
Através desse método nós podemos recuperar o que há no índice selecionado.

### IndexOf
```
lista.indexOf(object);
```
Serve para descobrir qual é a posição de tal objeto.

### Set
```
lista.set(i, e);
```
Serve para substituir um elemento por outro em determinada posição. Onde "i" é a posição e "e" é o elemento. Para que o resultado possa ser mais efetivo, nós podemos colocar, como posição, o método indexOf.

### Contains
```
lista.contains(object);
```
Verifica se existe ou não determinado elemento retornando true ou false.

### Clear
```
lista.clear();
```
Irá limpar toda a lista.

### IsEmpty
```
lista.isEmpty();
```
Nesse caso, ele irá apenas fazer a verificação se a lista está vazia ou não.

## Métodos com Collections

### Min
```
Collections.min(list);
```
Levando em consideração uma lista do tipo double, int ou qualquer outra forma numeral, com isso irá retornar o menor valor da lista selecionada.

### Max
```
Collections.max(list);
```
Levando em consideração uma lista do tipo double, int ou qualquer outra forma numeral, com isso irá retornar o maior valor da lista selecionada.

### Iterator
O iterator serve para verificar, em um laço de repetição, se há algum próximo elemento. Ele pode ser utilizado para fazer a soma em uma lista de números. Ele deve ser importado.
Exemplo:
```
Iterator<Double> iterator = notas.iterator(); //Lista notas
Double soma = 0d;

while(iterator.hasNext()){
    Double next = iterator.next();
    soma += next;
}
```
No caso dessa lógica, irá verificar se há mais elementos na lista. Caso tenha, a variável next irá receber o próximo valor através do iterator.next();

### Shuffle
Irá dispor os elementos de forma aleatória.
```
Collections.shuffle(list);
```

### Sort
Supomos que criamos uma classe gato que receberá vários valores sobre os gatos que estarão na lista. Para organizar em ordem alfabética, nós devemos implementar a interface Comparable nela e adicionar o compareTo. Somente após isso, na classe main que nós poderemos organizar.
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