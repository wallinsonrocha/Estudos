# Testes com Spring

# JUnit - Vanilla

## Implementação tradicional

1. Clicamos com o botão direito em nosso projeto e vamos até **Build Path**, e depois **Configure Build Path**.
2. Na aba **Libaries**, selecionamos **Classpath** e, na lateral, vamo em **Add Libary** e escolhemos o JUnit.
3. Após isso, basta configurá-la e finalizar clicando em **Apply and Close**.

## Recomendações

1. Para fazermos o uso do JUnit, recomenda-se usar uma classe cujo sufixo será Test. Exemplo: UserTest. Isso é comumente usando. 
2. Além disso, ela deverá ser criada em um pacote separado parar testes.
3. O nome dos métodos (tests), recomenda-se usar: ação+should+efeito+(opcional)when+condição:

```
public void depositShouldIncraseBalanceWhenPositiveAmount(){
    ...
}
```

4. Ao rodar a aplicação, devemos rodar ela como JUnit Test.

## Base

### @Test

Através dessa anotatio, indicamos ao JDK que o método será um test. E, dentro do método, especificamos o  seu funcionamento.

```
@Test
public void depositShouldIncraseBalanceWhenPositiveAmount(){
    double amount = 200.0;
    double expectedValue = 196.0;
    Account acc = new Account(1L, 0.0); // Nesse caso, no exemplo, era o id e o saldo.

    acc.deposit(amount);

    Assertion.assertEquals(expectedValue, acc.getBalance()); // Classe comumente usada para auxiliar nos testes.

}
```

Além disso, destaca-se lembrar do modelo AAA. Arrange (instanciar objetos), Act (executar as ações nescessárias) e Assert (declarar o que deveria acontecer).

## Padrão de projeto - Factory

Esse pad~rao é utilizado, geralmente, quando há muitos objetos que precisam ser instanciados. Ele pode ajudar a reduzir o tempo para criar uma novo objeto de forma padrão. Para isso, no pacote teste, nós implementamos um subpacote chamado factory. Lá nós iremos especificá-lo para que ele possa ser chamado de maneira mais rápida no nosso teste principal.

```
public class AcoountFactory{
    public static Account createEmpatyAccount(){
        return Account(1L, 0.0);
    }
}
```

Nós tambén podemos colocar os parâmetros, caso tenha. Ele poderá ser instanciado na nossa classe principal de testes.
```
@Test
... {
    ...
    Account acc = AccountFactory.createEmpatyAccount();
}
```

---

# JUnit com spring

Todos os teste executados deverão ocorrer no pacote de test de um projeto com springboot. **src/test/java**. Dentro dele, nós deveremos colocar o nome dos pacotes igual ao que fizemos no nosso ambiente normal.

Também é recomendado criar as classes com os mesmo nomes terminando com Tests. Se quisermos, por exemplo, criar uma classe de test para o ProductRepository, devemos por **ProductRepositoryTests**.

## Testes de Unidade repository

### @DataJpaTest

Usado para fazer os testes direcionados aos repositórios JPA. Os dados que serão analizados serão os que já podem ser implementados na inicialização do projeto, no caso o **import.sql**.

```
@DataJpaTest
public class ProductRepositoryTests{

    @Autowired
    private ProductRepository repository;

    @Test
    public void deleteShouldDeleteObjectWhenIdExists(){

        long exintingId = 1L;

        repository.deleteById(exintingId);

        Optional<Product> result = repository.findById(exintingId);
        Assertions.assertFalse(result.isPresent());
    }

}
```

## Fixtures

As fixtures pode ajudar o test a fazer coisas um pouco mais complexas. Para cada uma das anotações nós podemos executar algo. @BeforeAll, @AfterAll, @BeforeEach e @AfterEach. Geralmente é utilizado em testes mais complexos e grandes.

```
@DataJpaTest
public class ProductRepositoryTests{

    private long exintingId;

    @Autowired
    private ProductRepository repository;

    // Nesse caso o que tiver aqui será executada antes de cada test.
    @BeforeEach
    void setUp() throws Exception{
        exintingId = 1L;
    }

    @Test
    public void deleteShouldDeleteObjectWhenIdExists(){

        repository.deleteById(exintingId);

        Optional<Product> result = repository.findById(exintingId);
        Assertions.assertFalse(result.isPresent());
    }

}
```

Como antes de cada teste sera instanciado algo, não precisaremos colocá-la no escopo de cada test. Quando for a vez do objeto ser usado, ele será.