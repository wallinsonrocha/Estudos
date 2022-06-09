








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