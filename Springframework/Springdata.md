# Springdata
Usado como implementação ao JPA trabalhando, principalmente, na percistencia de dados.

## Instalação
Para iniciar o projeto, nós iremos no site do [spring initializr](https://start.spring.io/). Mas, dessa vez, nós iremos implementar a dependência **Spring Data JPA**, **MySQL Driver**, **Spring Web**, **Validation**. Caso esolha fazer testes, podemos instalar as dependências do H2.

![Mostrando as dependências](./Fotos/dependencias%20spring.png)

## JpaRepository
Quando vamos criar uma classe para ser considerada uma tabela, fazemos normalmente de acordo coma o mapeamento do Jpa. O SprintgData vai atuar na implementação dessas datas.

Criemos uma interface que irá extender JpaRepository. Nele será indicado qual será a tabela que criamos (a classe que criamos para isso) e o tipo de ID (identificador).
```
@Repository // Opcional
public interface UserRepository extends JpaRepository<User, Integer>{}
```

A partir desse momento iremos estar à disposição de vários métodos.
Quando criamos uma classe principal com o CommandLineRunner, nós iremos criar um obeto de a cordo com a interface que extendemos a JpaRepository.

```
@Component
public class StartApp implements CommandLineRuner{
    @Autowired
    private void run(String... args) throws Exception{
        ...
    }

}
```

## Conectando com o MySQL ou H2.
### MySQL
Para isso, no arquivo **apllication.properties**, adicionamos esse arquivo:
(Não esqueça de editar)
```
# usuário e senha de conexão com o banco de dados
spring.datasource.username=root
spring.datasource.password=root
# url de conexão do banco de dados
spring.datasource.url=jdbc:mysql://localhost:3306/blog?allowPublicKeyRetrieval=true&rewriteBatchedStatements=true&useSSL=false&useUnicode=yes&characterEncoding=UTF-8&useLegacyDatetimeCode=true&createDatabaseIfNotExist=true&useTimezone=true&serverTimezone=UTC

# apontamos para o JPA e Hibernate qual é o Dialeto do banco de dados
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
# deixamos o hibernate responsável por ler nossas entidades e criar as tabelas do nosso banco de dados automaticamente
spring.jpa.hibernate.ddl-auto=create
# configuração do Hibernate para reconhecer o nome de tabelas em caixa alta
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
# configurações de log
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true
```

ou

```
spring.jpa.hibernate.ddl-auto=update

spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/db_example
spring.datasource.username=springuser
spring.datasource.password=ThePassword
spring.jpa.properties.hibernate.jdbc.lab.non_contextual_creation=true
```

### H2
Criemos um arquivo chamado **application-test.properties**.
Nele colocaremos:
```
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=

spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

E, dentro do arquivo **application.properties**, adicionamos:
```
spring.profiles.active=test
spring.jpa.open-in-view=true
```

Após isso, recomenda-se criar uma classe para configuração. Criemos, pois, o pacote config com uma classe chamada TestConfig. Dentro dela, nós iremos identificar para o spring que ela é uma classe de configuração com a seguinte anotation: **@Configuration** e, para torná-la específica para as configurações de testes, colocaremos **@Profile("test")**, que é o mesmo nome que colocamos no profile de application.properties. Além disso, devemos implementar a interface **CommandLineRunner**.
```
@Configuration
@Profile("test")
public class TestConfig implements CommandLineRunner{
    @Autowired
    private UserRepository userRepository;

    @Override
    public void run(String... args) throws Exception{
        User u1 = new User(...).;
        User u2 = new User(...);

        userRepository.saveAll(Array.asList(u1, u2));
    }
}
```

Dentro dela irá as implementações de dados. Nesse caso, a criação de usuários.

## Recomendação para criação de pacotes.
Recomenda-se a criação dos seguintes pacotes para que haja melhor manutenção do código e organização:
1. Config - Configuração
2. Entities - As tabelas
3. Repositories - Para o repositório
4. Services - Para adicionar as services
5. Controllers - Controles.

## Estrutura básica
### Entity
```
@Entity
@Table(name = "tb_value")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    private String password;
}
```

### Repository
```
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

### Service
```
@Service
public class UserService {

    @Autowired
    private UserRepository repository;

    public List<User> findAll(){
        return repository.findAll();
    }
    
    public User findById(@PathVariable Long id){
        Optional<User> obj = repository.findById(id);
        return obj.get();
    }
}
```

### Controller
```
@RestController
@RequestMapping(value = "/users")
public class UserController {
    @Autowired
    private UserService service;

    @GetMapping
    public ResponseEntity<List<User>> findAll(){
        User u1 = new User(null, "Carlos", "carlos@email.com", "123456");
        List<User> list = service.findAll();
        return ResponseEntity.ok().body(list);
    }

    @GetMapping(value = "/{id}")
    public ResponseEntity<User> findById(@PathVariable Long id){
        User obj = service.findById(id);
        return ResponseEntity.ok().body(obj);
    }
}
```

---

## Algumas anotations 
### @RestController
Serve para identificar a classe como Bean do tipo controller. Geralmente é colocado para gerar as dependências quando necessário. É colocado na classe principal da nossa aplicação. Dentro da classe que recebe essa anotation é colocada os GetMapping. Ela irá fazer as chamadas no service que irá fazer a comunicação com o repository.

Através dela ocorrerá essa comunicação. Controller -> Services -> Repositories.

### @CrossOrigin()
Um identificador para permitir que a classe seja acessada de qualquer fonte.
```
@RestController
@CrossOrigin(origin = "*", maxAge = 3600)
public class MyAppController{
    ...
}
```

### @RequestMapping()
RI a nível de classe. Serve para identificar por onde nós podemos acessar o recurso criado. Esse será o caminho base para que os demais mappings possam ser acessados. Quando há algum método, seja ele get, post... e não houver alguma direção para ele, eles serão encaminhados para cá e dará prosseguimento ao processo.
```
@RestController
@CrossOrigin(origin = "*", maxAge = 3600)
@RequestMapping("/direcao")
public class MyAppController{
    ...
}
```

### @GetMapping()
Serve para identificar tal método através do direcionamento da url. Caso seu objetivo seja ele ir para o RequestMapping, não precisamos identificar a sua direção.
```
@GetMapping("/direcao")
public String index(){
    return "Hello World";
}
```

Nós ainda podemos fazer um tipo de identificação para conseguir um ID de acordo com a url.
Na classe que estará o **Service**, implementamos:
```
public List<User> = findById(Long id){
    Optional<User> obj = repository.findById(id);
    return obj.get();
}
```

Na classe que estará o Controller:
```
@GetMapping("/{id}")

public ResponseEntity<User> findById(@PathVariable Long id)  { //ResponseEntity pertence ao spring
    User obj = service.findById(id);
    reuturn ResponseEntity.ok().body(obj); // Para a requisição ser exposta no body.
} 
```

Esse **@PathVariable** serve para fazer a comunicação no {id}. Quando nós colocarmos na url o id, ele irá reconhecer e implementar.

### @PostMapping
Através do postmapping somos capazes de criar algo dentro do nosso banco de dados. Geralmente, para que isso ocorra de maneira correta, quando usamos o postman, deve-se retornar o código 201. E, para isso, devemos implementar algums métodos.
**Service**
```
public User insert(User obj){
    reuturn repository.save(obj);
}
```

**Controller**
```
@PostMapping
public ResponseEntity<User> insert(@RequiredBody User obj){
    obj = service.insert(obj);
    URI uri = ServletUriComponentsBuilder.fromCurrentRequest().path("/{id}")
        .buildAndExpand(obj.getId()).toUri();
    return ResponseEntity.created(uri).body(obj);
}
```

### @DeleteMapping
Através dele podemos deletar algum usuário. No arquivo de service, onde faz a chamada do repositório, há várias chamadas como o deleteById.
```
@DeleteMapping(value = "/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id){
    service.delete(id);
    return ResponseEntity.noContent().build();
}
```

Caso tenha algo ligado àquele id, como no caso uma ordem de pedido, a deleção dele não será possível. Para isso, a ligação a ele deverá ser removida.

### @PutMapping
Para fazer atualizações em algum usuário.
**Services**
```
public User update(Long id, User obj){
    User entity = repository.getOne(id);
    updateData(entity, obj);
    return repository.save(entity);
}

private void updateData(User entity, User obj){
    entity.setName(obj.getName());
    ...
}
```

**Controllers**
```
@PutMapping(value = "/{id}")
public ResponseEntity<User> update(@PathVariable Long id, @RequestBody User obj){
    obj = service.update(id, obj);
    return ResponseEntity.ok().body(obj);
}
```

### @Service
Identificação da classe que fará a comunicação entre o Repository e o Controller.

### @Transactional
Colocada no método que podem fazer modificações no banco de dados como o save, delete... Ele garante que, se houver algum erro, ele volta às alterações antigas.
```
@Service
public class EstacionamentoService{
    @Transactional
    public EstacionamentoModel save(){
        ...
    }
}
```

### @Transient
Impede o Jpa de interpretar algum atributo. Geralmente usado para pular erros.

### @ManyToOne
Essa anotation serve para indicar a alguma propriedade que ele receberá muitas outras informações. Por isso se chama muitos para um.

Supomos que fora criado uma classe para receber ordens de pedidos. Nele nós teremos as propriedades como o momento do pedido (o java pode usar o a classe Instant), o Id e o cliente. Este pode pedir várias coisas. Então, para que isso seja explicito, a anotation em cima da propriedade cliente irá ajudar.

Então, para isso, iremos passa a anotation em cima dessa propriedade para que a tabela de Usuários faça um relacionamento com a tabela de ordens.

Com ela, será criada a chave estrangeira. Além disso, com o **@JoinColum** podemos  essa chave estrangeira. Esta chave estrangeira faz um relacionamento entre duas tabelas. Através dela (foreign key) especificamos quais são as colunas que a tabela está relacionada.

```
public Class Order implements Serializeble{
    ...
    @ManyToOne
    @JoinColum(name='client_id')
    private User client;
    ...
}
```

### @OneToMany
Com essa anotation nós reforçamos essa ligação. Para que ela seja usada, nós devemos mapear ela de acordo com o atributo que contém a anotation @ManyToOne
```
@OneToMany (mappedBy = 'client')
private List<Order> orders = new ArrayList<>();
```

### @JsonIgnore
Vem, geralmente, nas prorpiedades que possue o OnetoMany ou ManyTOne. Em uma aplicação, quando há uma propriedade chamando a outra, a chamada pode ficar infinita. É aí que o JsonIgnore entra. Onde ele estiver, a chamada dele será interrompida.

---

## Anotations do Spring Validation
Elas vão, em sua maioria, em atributos. Servem para verificá-las.

### NotBlack
Para verificar se o valor não está vazio.

### Size()
Para verificar se o campo tem mais ou menos caracteres. Para isso, identificamos com "max = numero" ou "min = numero".
