# PHP

## Declaração de variáveis.
Todas as variáveius em PHP devem começar com **$**. Além disso, diferente da linguagem C e Java, não somos obrigados a especificar o tipo dela.
```
$variavel = valor;

$numero = 12;
$nome = 'Strings';
$numero = 12.5;
$logicoTrue = True;
$logicoFalse = False;
```

## Constantes
As constantes servem para valores que nunca irão mudar.
```
define("NOME_DA_CONSTANTE", valor)
```

As constantes podem ser de qualquer tipo. Inteiro, float, string ou boolean. Elas são escritas, preferencialmente, com letras maiúsculas.

Além dessa forma, ela poder ser declarada assim como no javascript:
```
const PI = 3.14
```

## Concatenação
Serve para ligar textos em uma String para expor na tela.
```
$nome = 'Wallinson';

echo 'Meu nome é '. $nome;
```
Ela é feita com `.`.

## Condicionais
As condicionais servem para verificar uma determinada condição para que tal código seja executado.

### If e Else
```
$numero = 2;

if($numero < 4){
    echo 'Numero menor que 4';
} else{
    echo 'Numero maior que 4';
}
```

### Switch
```
$letra = 'a';

switch($letra){
    case 'a':
        echo 'Letra a';
        break;

    case 'b':
        echo 'Letra b'
        break;

    case 'c':
        echo 'Letra c'
        break;

    default:
        echo 'Não é nem a, nem b, nem c';
}
```

## Laços de repetição
Os laços são repetções que repetem determinadas linhas de código até que determinado parâmetro seja satisfeito.

### while e do while
```
$i = 0;

while($i != 5){
    $i = 1 + $i;
    echo $i;
}

do{
    $i = 1 + $i;
    echo $i;
} while ($i == 0);
```
 A diferença entre o while e o do while está quando a condição é verificada. Com o `while` a condição é vista primeira. Por sua vez o `do while` faz a verificação no final. Observe que, no exemplo acima o **do while** entraria em um looping eterno uma vez que a sua condição é verificada no final.

 ### for
O laço for é um laço semelhante ao while, porém menos flexível.
```
for(variável=valor; condição; incremento ou decremento){
    //Lógica
}

Exemplo:
for($i = 0; $i <= 10; $i++){
    echo $i;
}

// Lê-se: Enquanto a variável i for menor igual a 10, acrescente 1.
```

### foreach
Os foreach são usados, geralmente, em arrays. Na próxima anotação veremos isso. Por hora, não precisamos nos preocupar.
```
//FOREACH => Tradicional
foreach($array as $arrays){
    //Lógica
}

//FOREACH => Para percorrer chave e valor em um vetor
foreach($array as $idx => $nome_array){
    //Lógica
}
```

## Casting de tipos