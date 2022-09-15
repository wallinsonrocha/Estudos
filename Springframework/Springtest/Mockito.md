# Mockito

O mockito irá nos auxiliar na criação de objetos mockados para realizar os nossos testes. Para isso, em nossa classe que criamos para o controller, adicionamos a seguinte anotation: **@ExtendWith(MockitoExtension.class)**.
Exemplo:

```
@ExtendWith(MockitoExtension.class)
public class CalculatorControllerTest{
    ...
}
```

## @InjectMocks

Ao chamar o controller para a execução dos nossos testes, devemos lembrar que eles fazem conexões com o repository. Então, para os nossos objetos controllers, devemos por a anotation InjetcMocks ao invés do Autowired.

```
@InjectMocks
private ClassController classController;
```

Quando ele for testado irá executar a sua função da maneira que foi programado para executar. Supomos que na classe controller há uma função sum para somar números.

```
void shouldSumNumber(){
    Double result = classController.sum(3.5, 3.5);
    Assertions.assertEquals(7, result);
}
```