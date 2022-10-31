# Python

## Manipulação de Strings

### Maiúscula, minúscula e título

```
curso = "pYthon"

print(curso.upper())
// PYTHON

print(curso.lower())
// python

print(curso.title())
// Python
```

### Eliminar espaços em branco
```
curso = " Python "

print(curso.strip())
// "Python"

print(curso.lstrip())
// "Python "

print(curso.rstrip())
// " Python"
```

### Junções e centralização
```
curso = "Python"

print(curso.center(10, "#"))
// "##Python##"
// Nesse caso ele irá preencher os espaços até que tenha 20 caracteres.

print(".".join(curso))
// "P.y.t.h.o.n"
```

## Interpolação de variáveis

É uma forma de colocar valores de variáveis em uma string. Um dos modelos mais usados é o "f-string".

```
nome = "Guilherme"
idade = 28
profissao = "Programador"
linguagem = "Python"

print(f"Olá, me chamo {nome}. Eu tenho {idade} anos de idade, trabalho como {profissao} e programo em {linguagem}.")
```

Podemos, também, fazer formatações com ela:
```
PI = 3.14159

print(f"Valor de PI: {PI:.2f}")
```

Nesse caso estou limitando o número de casas decimais.

## Fatiamento 

Nós podemos dividir uma string de acordo com o array da variável. Podemos selecionar um caractere ou um intervalo.
```
// variavel[inicio:fim:intervalo] ou variavel[posicao]

nome = "Wallinson Ribamar Nascimento Rocha"

nome[0]
//W

nome[:9]
// Wallinson

nome[10:17]
//Ribamar

nome[10:]
// Ribamar Nascimento Rocha

nome[10:17:2]
//Rbmr

nome[:: -1]
// ahcoR otnemicsaN ramabiR nosnillaW
```

## String multipla

Utilizada, geralmente, para escrever textos personalizados e longos.

```
print(
    """
    ############# MENU ###############

    1 - Depositar
    2 - Sacar
    0 - Sair

    ##################################

            Obrigado por usar o sistema
    """
)
```