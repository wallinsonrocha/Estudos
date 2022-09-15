# MockBean - Testes nos services.

Após a criação de uma classe que irá fazer parte do nosso banco de dados, na pasta de tests da nossa aplicação, iremos criar uma classe que estará responsável pelos testes.

Após isso, nessa mesma classe, iremos por algumas anotações para o seu funcionamento.

```
@RunWith(SpringRunner.class)
public class ClasseServiceTest{

    @Autowired
    ClasseService classeService;

    @Test
    public void daysCalculatorShouldReturn10WhenParametreSpicify(){
        String name = "Wallinson";
        int days = classService.daysCalculatorWithDatabase(name);

        Asseritions.assertEquals(days, 10);
    }


}
```

## Acesso ao service

Para termos esse acesso, usaremos o MockBean. Para isso, iremos intanciá-lo dentro da nossa aplicação. Não podemos esquecer que, por ser um bean de test, nós devemos especificá-lo com a anotação **@TestConfiguration** para que ele seja usado apenas em ambientes de testes.

```
@RunWith(SpringRunner.class)
public class ClasseServiceTest{

    @TestConfiguration
    static class ClasseServiceTestConfiguration{
        @Bean
        public ClasseService classeService(){
            return new ClasseService();
        }
    }

    ...

}
```

Nós iremos precisar, também, retornar uma instancia do repository. Então, para isso, nós iremos utilizar o Mockito na anotação Before. Mas, para que ele seja usado, nós iremos precisar adicionar um **@MockBean** de repository na nossa aplicação. Ele será necessário para implementarmos no "when".

```
 @MockBean
    ClasseRepository classeRepository;

@Before
public void setup(){
    Classe classeModel = new Classe(...)

    Mockito.when(classeRepository.findByName(classeModel.getName()))
        .thenReturn(classeModel);
}
```