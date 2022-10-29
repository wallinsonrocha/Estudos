# Python

## Funções
Através das funções podemos resumir várias linhas de código. A sua declaração é simples.

```
def exibir_mensagem():
    print("Hello World")

exibir_mensagem() # Hello World
```
Nesse caso a função será expor uma mensagem.

Além desse formato, podemos por atributos para serem analizados e retornar um resultado.
```
def exibir_mensagem(nome="Pedro"):
    print(f"Seja bem vindo {nome}!")

exibir_mensagem(nome="Wallinson") # Seja bem vindo Wallinson!
exibir_mensagem("Wallinson") # Modo direto.
```

Caso não seja definido, podemos colocar um valor padrão - no caso acima, "Pedro". Isto é opcional.

## Exemplos
Veja abaixco outros exemplos

```
def calcular_total(numeros):
    return sum(numeros)

calcular_total([10, 20, 34]) # 64
```

```
def exibir_carro(marca, modelo, ano, placa):
    print(f"Carro : {marca}, {modelo}, {ano}, {placa}")

exbir_carro(marca="Fiat", modelo="Palio", ano=1999, placa="ABC-1234")

Ou poderemos passar um dicionário como argumento.

exbir_carro(**{marca="Fiat", modelo="Palio", ano=1999, placa="ABC-1234"})
```

> ** recebe dicionário e * recebe tupla.