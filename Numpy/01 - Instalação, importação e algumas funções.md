# Numpy

## Instalação e importação
### Instalação
```
pip install numpy
```
### Importar Numpy
```
import numpy as np
```

## Criar arrays
```
a = np.array([1, 2, 3, 4, 5, 6])
```

## Algumas funções:
### .reshape()
Transforma a matriz em um vetor coluna. Os seus parâmetros são `reshape(coluna, elementos_por_coluna`). O número de elementos deve ser, obrigatóriamente, dentro dos limites do array original.
```
a.reshape(2, 3)
# [
  [1, 2, 3]
  [4, 5, 6]
]
```
### .T
Transposição de matriz. Para transforma a matriz em transposta.

### np.sum()
Ele irá fazer somas de acordo com o parâmetro passado. O seu parâmetro é `np.sum(array, axis=<1 ou 2>)`
Nesse `axis`, que é opcional, poderemos por.

- 1: Soma ao longo das linhas (vertial). Ele irá retornar um array com as somas.
- 2: Soma ao logo das colunas (horizontal). Ele irá retornar um array com as somas.

### np.mean()
Ele irá fazer a média de acordo com o parâmetro passado. O seu parâmetro é `np.mean(array, axis=<1 ou 2>)`
Nesse `axis`, que é opcional, poderemos por.

- 1: Média ao longo das linhas (vertial). Ele irá retornar um array com as médias.
- 2: Média ao logo das colunas (horizontal). Ele irá retornar um array com a soma.

### np.median()
Ele irá tirar a mediana de acordo com o parâmetro passado. O seu parâmetro é `np.mean(array, axis=<1 ou 2>)`
Nesse `axis`, que é opcional, poderemos por.

- 1: Mediana ao longo das linhas (vertial). Ele irá retornar um array com a mediana.
- 2: Mediana ao logo das colunas (horizontal). Ele irá retornar um array com a mediana.

### np.where()
Ele ira retornar uma resposta boolean de acordo com a condição que passamos. A sua sintaxe é `np.where(var)`.
```
cond = array % 2 == 0
i, j = np.where(cond)
```

Nesse caso ele irá retornar as variáveis i e j representando, respectivamente, linha e coluna. Não obstante, podemos utilizar para outras finalidades, também.
```
