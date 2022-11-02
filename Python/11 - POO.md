# Python

## Programação Orientada a Objetos
Programação orientada a objetos é um paradigma de programação baseado no conceito de "objetos", que podem conter dados na forma de campos, também conhecidos como atributos, e códigos, na forma de procedimentos, também conhecidos como métodos.

## Classe e Objeto
Uma classe define as características e comportamentos de um objeto, porém não conseguimos usá-la diretamente. Já os objetos podemos usá-los e eles possuem as características e comportamentos que foram definidos nas classes.

Declaração:

```
class Cachorro:
    # Construtor do atributos do cachorro
    def __init__(self, nome, cor, acordado=True):
        self.nome = nome
        self.cor = cor
        self.acordado = acordado
    
    # Métodos do cachorro
    def latir(self):
        print("Auau")
    
    def dormir(self):
        self.acordado = False
        print("Zzzz...")
```

Criando um objeto cachorro:
```
cao_1 = Cachorro("chappie", "amarelo")
cao_1.latir()
```

## Herança 
Através da herança podemos extender uma determinada classe para outra. Diferente de algumas linguagens que utiliza **extends** para extender, no python é utilizado "(Classe)". Nós podemos extender mais de uma classe (herança múltipla).
```
class Mamifero:
    def __init__(self, pelo, acordado=True):
        self. pelo = pelo
        self.acordado = acordado

class Cachorro(Mamifero):
    def latir(self):
        print("Auau")
```

Observe que na classe cachorro não foi necessário criar constutores. Isso é possível por conta da herança. Através da herança os atributos de Mamífero foram herdados. Porém, se necessário criar algum atributo específico, isso também é possível. Exemplo:
```
class Mamifero:
    def __init__(self, pelo, acordado=True):
        self. pelo = pelo
        self.acordado = acordado

class Cachorro(Mamifero):
    def __init__(self, raca):
        self.raca = raca

    def latir(self):
        print("Auau")
```

Para acessar as características da classe herdada usamor o **super()** e, no construtor da classe que recebeu a herança, colocamos uma **\*\*kw**. Há uma outra forma que podemos utilizar sem kw.
```
class Mamifero:
    def __init__(self, pelo, acordado=True):
        self. pelo = pelo
        self.acordado = acordado

class Cachorro(Mamifero):
    def __init__(self, raca, **kw):
        self.raca = raca
        super().__init__(**kw)

    #ou 

    def __init__(self, raca, pelo):
        self.raca = raca
        super().__init__(pelo=pelo)

    def latir(self):
        print("Auau")

sansao = Cachorro(pelo="amarelo", raca="Vira lata")
```

## __str__
Irá definir uma string build para retornar o que queremos exibir na tela. Utilizado, geralmente, para tornar mais visível as características de uma classe.
```
class Mamifero:
    def __init__(self, pelo, acordado=True):
        self.pelo = pelo
        self.acordado = acordado

class Cachorro(Mamifero):
    def __init__(self, raca, **kw):
        self.raca = raca
        super().__init__(**kw)

    def __str__(self):
        return "Cachorro"
```

## Encapsulamento
No python, diferente do Java, não há uma palavra que defina um atributo como público ou privado. Como convenção colocamos um "_" (private) ou "__" (protected) para definí-lo como privado. Isso serve para atributos e para métodos.
```
class Mamifero:
    def __init__(self, pelo, acordado=True):
        self._pelo = pelo
        self._acordado = acordado
```

### Getters and Setters
Para criá-los, usamos um decorador chamado **@property** (para o getter) e o **@nome.setter** (para setter). Ele irá ser colocado junto a um método.
```
class Produto:
    def __init__(self, nome, preco):
        self.nome = nome
        self.preco = preco

    @property
    def preco(self):
        return self._preco

    @preco.setter
    def nome(self, valor):
        self._preco = valor
```