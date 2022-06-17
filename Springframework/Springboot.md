# Springboot
## Inicialização
Para iniciar o projeto, devemos gerá-lo através site do [spring initializr](https://start.spring.io/).
Após isso, basta extraí-lo e abrir com o Intellij (a ide que utilizo).

 ## Criação e chamada de classe.
 Com o springboot, a chamada de classe e a sua criação é muito diferente.
 
 1. Criemos, primeiro, uma classeque funcionará como o main e implementaremos a interface CommandLineRunner. Além disso, para idenficá-lo, devemos por uma anotation **@Component**:
 ```
 @Component
 public class MyApp implements CommandLineRunner{
    ...
 }
 ```

 2. Após isso, ele irá pedir para implementar o método run(). Dentro dele irá os nossos códigos.
 ```
 @Component
 public class MyApp implements CommandLineRunner{
    @Override
    public void run(String... args) throws Exception{
        ...
    }
 }
 ```

 3. Supomos que criamos uma classe chamada calculadora. Através dessa classe nós iremos por um método soma. Além disso, ao criá-lo, devemos identificá-lo com a anotation **@Component**:
 ```
 @Component
 public class Calculadora{
    public int soma(int a, int b){
        return a+b;
    }
 }
 ```

 4. Para chamá-lo na nossa classe MyApp, não será necessário criar um "new Calculadora". Nós apenas precisaremos idenficicá-lo com o **@Autowired**. Dessa forma a classe será instanciada.
 ```
 @Component
 public class MyApp implements CommandLineRunner{
    @Autowired
    private Calculadora calculadora;
    @Override
    public void run(String... args) throws Exception{
        System.out.println("Soma: " + calculadora.soma(5, 4))    ;
    }
 }
 ```

---

 ## Bean
 O Beans é utilizado quando não temos acesso ao código fonte. Um exemplo é a biblioteca do Google **Gson**. Geralmente, por conta disso, recomenta-se criar uma classe Beans que irá abrigar essa biblioteca.
 ```
 @Configuration
 public class Beans{
    @Bean
    public Gson gson(){
        return new Gson();
    }
 }
 ```

 Pode ser que, em alguns momentos, nós precisaremos usar a anotation **@Configuration** para que isso seja aceito. Então, logo em seguida, podemos chamá-la em nossa aplicação com o **@Autowired**.

## Value
O value é utilizado para atribuir valores de acordo com o que tem no arquivo **application.properties**.
```
@Value("${nome:NoReply}")
private String nome;
@Value("${email:NoReply}")
private String email;
@Value("${telefones:NoReply}")
private List<Long> telefones = new ArrayList<>(Array.asList(new Long[]{1195678124L}));
```

Para que o valor colocado na anotação dê certo, ela deve ser escrita corretamente. Caso contrário, o valor que colocamos como padrão (depois dos dois pontos), irá ser posto como valor.

Caso haja algum valor que foi colocado (como nós podemos ver na Lista de telefones), ele será substituído.

## ConfigurationProperties
Nós podemos simplificar o uso do value utilizando o **ConfigurationProperties** identificando o prefixo da propriedade.
**Arquivo application.properties**
```
remetente.nome= ...
remetente.email= ...
remetente.telefones= ..., ...
```

**Classe remetente**
```
@Configuration
@ConfigurationProperties(prefix = "remetente")
public class Remetente{
   private String nome;
   private String email;
   private List<Long> telefones;
}
```

---

## Scope - Singleton e Prototype.
O Scope refere-se à criação de objetos. Com o Singleton indicamos que aquela classe cria apenas um objeto. Este vem como valor padrão sema necessidade de colocar um scope. Poŕem, definindo o Scope de uma classe como prototype, podemos criar vários objetos.
Exemplo:
**Classe Remetente**
```
@Configuration
public class Beans{
   @Bean
   @Scope("prototype")
   public Remetente remetente(){
      System.out.println("CRIANDO UM OBJETO REMETENTE");
      Remetente remetente = new Remetente();
      remetente.setEmail("noreply@dio.com.br");
      remetente.setNome("Digital Innovation One");
      return remetente;
   }
}
```

**Classe Sistema Mensagem**
```
@Component
public class SistemaMensagem{
   @Autowired
   private Remetente noreply;
   @Autowired
   private Remetente techTeam;

   public void enviarConfirmacaoCadastro(){
      ...
   }
   public void enviarMensagemBoasVindas(){
      techTeam.setEmail("tech@dio.com.br");
      ...
   }
}
```

Nesse caso, o objeto techTeam terá o seu nome alterado. Antes, ao mudar o techTeam, o noreply também era alterado.

---