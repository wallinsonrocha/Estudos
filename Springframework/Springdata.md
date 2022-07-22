# Springdata

- [Instalação](#instalação)
- [JpaRepository](#jparepository)
- [Conectando com Banco de dados](#conectando-com-banco-de-dados)
    - [MySQL](#mysql)
    - [H2](#h2)
- [Recomendação para criação de pacotes](#recomendação-para-criação-de-pacotes)
- [Estrutura básica](#estrutura-básica)
    - [Entity](#entity)
    - [Repository](#repository)
    - [Service](#service)
    - [Controller](#controller)
- [Algumas anotations](#algumas-anotations)
    - [@RestController](#restcontroller)
    - [@CrossOrigin()](#crossorigin)
    - [@RequestMapping()](#requestmapping)
    - [@GetMapping()](#getmapping)
        - [FindById](#findbyid)
        - [FindAll](#findall)
    - [@PostMapping](#postmapping)
    - [@DeleteMapping](#deletemapping)
    - [@PutMapping](#putmapping)
    - [@Service (anotação)](#service-anotação)
    - [@Transactional](#transactional)
    - [@Transient](#transient)
    - [@ManyToOne](#manytomany)
    - [@JsonIgnore](#jsonignore)
    - [@Column](#column)
    - [@Table](#table)
- [Tratamento de exceções](#tratamento-de-exceções)
    - [Início](#início)
    - [Configuração de cada arquivo](#configuração-de-cada-arquivo)
        - [DatabaseException](#databaseexception)
        - [NotFoundException](#notfoundexception)
        - [StandardError](#standarderror)
        - [ResoucerExceptionHandler](#resoucerexceptionhandler)
    - [Correções nos Services](#correções-nos-services)
        - [delete](#delete)
        - [update](#update)
        - [findById](#findbyid-correções)
- [Paginação](#paginação)
- [Banco de dados NoSQL](#banco-de-dados-nosql)
    - [Conectando o mongodb](#conectando-o-mongodb)
    - [Criação](#criação)
    - [Interações com o banco de dados](#interações-com-o-banco-de-dados)
        - [Obtendo um usuário por id](#obtendo-um-usuário-por-id)


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

## Conectando com Banco de dados

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

## Recomendação para criação de pacotes

Recomenda-se a criação dos seguintes pacotes para que haja melhor manutenção do código e organização:

1. Config - Configuração
2. Entities - As tabelas
3. Repositories - Para o repositório
4. Services - Para adicionar as services
5. Controllers ou Resources - Controles.
6. Exceptions - Para tratar exeções.

Caso seja NoSQL, no lugar de Entities, colocamos "Domain".

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

#### FindById

**Service**

```
public List<User> = findById(Long id){
    Optional<User> obj = repository.findById(id);
    return obj.get();
}
```

**Controller**:

```
@GetMapping("/{id}")

public ResponseEntity<User> findById(@PathVariable Long id)  { //ResponseEntity pertence ao spring
    User obj = service.findById(id);
    reuturn ResponseEntity.ok().body(obj); // Para a requisição ser exposta no body.
} 
```

#### FindAll

**Service**

```
public List<User> findAll(){
        return repository.findAll();
    }
```

**Controller**:

```
@GetMapping
    public ResponseEntity<List<User>> findAll(){
        List<User> list = service.findAll();
        return ResponseEntity.ok().body(list);
    }
```

Esse **@PathVariable** serve para fazer a comunicação no {id}. Quando nós colocarmos na url o id, ele irá reconhecer e implementar.

### @PostMapping

Através do postmapping somos capazes de criar algo dentro do nosso banco de dados. Geralmente, para que isso ocorra de maneira correta, quando usamos o postman, deve-se retornar o código 201. E, para isso, devemos implementar algums métodos.

**Service**

```
public User insert(User obj){
    return repository.save(obj);
}
```

Há uma outra forma de criar o service através de uma classe DTO.

```
public UserDTO insert(UserDTO dto){
    User entity = new User();
    
    CopyDtoToEntity(dto, entity);

    entity = respository.save(entity);
    return new UserDTO(entity);
}

...

private CopyDtoToEntity(UserDTO dto, User entity){
    entity.setName(dto.getName());
    entity.setDescription(dto.getDescription());
    ...

    // Considerando que as categorias no UserDTO foram configuradas
    entity.getCategories().clear();
    for(CategoryDTO catDTO : dto.getCategory()){
        Category category = categoryRepository.getOne(catDTO.getId);
        entity.getCategories().add(category);
    }
}
```

**Controller**

```
@PostMapping
public ResponseEntity<User> insert(@RequestBody User obj){
    obj = service.insert(obj);
    URI uri = ServletUriComponentsBuilder.fromCurrentRequest().path("/{id}")
        .buildAndExpand(obj.getId()).toUri();
    return ResponseEntity.created(uri).body(obj);
}
```

### @DeleteMapping

Através dele podemos deletar algum usuário. No arquivo de service, onde faz a chamada do repositório, há várias chamadas como o deleteById.

**Service**

```
public void delete(Long id){
    repository.deleteById(id);
}
```

**Controller**

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

Há uma outra forma de criar o service através de uma classe DTO.

```
public UserDTO update(Long id, UserDTO dto){
    try{
        User entity = new User();
        
        CopyDtoToEntity(dto, entity);

        entity = respository.save(entity);
        return new UserDTO(entity);
    } catch(EntityNotFoundException e) {
        throw new ResourceNotFoundException("Id not found " + id);
    }
}

...

private CopyDtoToEntity(UserDTO dto, User entity){
    entity.setName(dto.getName());
    entity.setDescription(dto.getDescription());
    ...

    // Considerando que as categorias no UserDTO foram configuradas
    entity.getCategories().clear();
    for(CategoryDTO catDTO : dto.getCategory()){
        Category category = categoryRepository.getOne(catDTO.getId);
        entity.getCategories().add(category);
    }
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

### @Service (anotação)

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

Com essa anotation nós reforçamos essa ligação. Para que ela seja usada, nós devemos mapear ela de acordo com o atributo que contém a anotation @ManyToOne. Exemplo feito na classe do Cliente. Obs: O many to one e o one to many se completam.

```
@OneToMany (mappedBy = 'client')
private List<Order> orders = new ArrayList<>();
```

> Obs.: Nas coleções nós não criamos Setters nem adicionamos ele no Construtor. A coleção que me refiro è a lista que foi instanciada.

### @ManyToMany

Essa anotação serve para estabelecer uma relação de muitos para muitos.

Junto dela, em um elemento da relação que escolhemos, devemos por a anotatio **@JoinTable**. Nela especificaremos o nome da tabela e os parâmetros de relação com a outra tabela.

```
public class Product ...
...
@ManyToMany
@JoinTable(name = "tb_product_category", 
    joinColumns = @JoinColumn(name = "product_id"),
    inverseJoinColumns = @JoinColumn(name = "category_id"));
private Set<Category> categories = new HashSet<>();
```

```
public class Category ...
...
@ManyToMany(mappedBy = "categories");
private Set<Product> products = new HashSet<>();
```

No curso do Nélio Alves, para adicionar elementos em alguns dos HashSets, na classe de Test (onde estava rodando a aplicação), ele fez:

```
p1.getCategories().add(cat2);
```

### @JsonIgnore

Vem, geralmente, nas prorpiedades que possue o OnetoMany ou ManyTOne. Em uma aplicação, quando há uma propriedade chamando a outra, a chamada pode ficar infinita. É aí que o JsonIgnore entra. Onde ele estiver, a chamada dele será interrompida.

---

### @Column

O column é colocado nos demais atributos de alguma entidade. Através dela nós podemos definir o nome que irá aparecer no banco de dados, podemos definir se ela irá ser ou não nula, além disso, podemos definir que tipo de valor ela irá receber.

```
@Column(name = "first_name", nullable = "false", columnDefinition = "TEXT")
private String firstName;
```

Normalmente a arquitetura do Java já diz se ele será nulo e que tipo de variável ele poderá receber. Mas, há algumas recomendações específica para o trabalho com datas, especialmente com o instante para que possamos trabalhar com o formato ISO 8601. O formato que irá ser apresentado instrui o banco de dados a guardar o atributo no formato UTC.

```
@Column(columnDefinition = "TIMESTAMP WITHOUT TIME ZONE")
private Instante date;
```

Além dela, outra forma de trabalhar é com os textos. Nós definimos o tipo de coluna para "text" para que sejam aceitos mais do que 255 caracteres. Geralmente é utilizado para descrições.
```
@Column(columnDefinition = "TEXT")
private String description;
```

### @Table

Utilizado, geralmente, para mudar o nome da entidade a fim de que ela não seja o mesmo da Classe Java.
```
@Table(name = "studentData")
public class Student{
    ...
}
```

## Tratamento de exceções

Ao criar as funções do CRUD, em alguns momentos poderá haver um erro ao requisitar um get, push ou qualquer outra coisa da nossa API. Por conta disso, precisamos fazer um tratamento para isso.

### Início

Para iniciar, devemos, dentro da pasta Exception, criar alguns arquivos e uma pasta. Os arquivos são:
1. DatabaseException - que irá extender Runtime Exception.
2. NotFoundException - que também irá extender de Runtime Exception.
3. Resoucer - que terá alguns recursos que usaremos;
4. ResoucerExceptionHandler - que estará dentro da pasta Resoucer.
5. StandardError - que também estará dentro da pasta Resoucer.

## Configuração de cada arquivo:

### DatabaseException
```
public class DatabaseException extends RuntimeException{
    private static final long serialVersionUID = 1L;

    public DatabaseException(String msg){
        super(msg);
    }
}
```

### NotFoundException
```
public class NotFoundException extends RuntimeException{
    private static final long serialVersionUID = 1L;

    public NotFoundException(Object id){
        super("Not found resoucer. Id" + id);
    }
}
```

### StandardError
```
public class StandardError implements Serializable {
    private static final long serialVersionUID = 1L;

    private Instant timestamp;
    private Integer status;
    private String error;
    private String message;
    private String path;

    public StandardError() {
    }

    public StandardError(Instant timestamp, Integer status, String error, String message, String path) {
        this.timestamp = timestamp;
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    public Instant getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Instant timestamp) {
        this.timestamp = timestamp;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    public String getError() {
        return error;
    }

    public void setError(String error) {
        this.error = error;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }
}
```

### ResoucerExceptionHandler
```
// Isso indica que essa classe irá tratar erros especiais.
@ControllerAdvice
public class ResourceExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<StandardError> notFound(NotFoundException e, HttpServletRequest request){
        String error = "Resoucer not found";
        HttpStatus status = HttpStatus.NOT_FOUND;
        StandardError err = new StandardError(Instant.now(), status.value(), error, e.getMessage(), request.getRequestURI());
        return ResponseEntity.status(status).body(err);
    }

    @ExceptionHandler(DatabaseException.class)
    public ResponseEntity<StandardError> database(DatabaseException e, HttpServletRequest request){
        String error = "Database error";
        HttpStatus status = HttpStatus.BAD_REQUEST;
        StandardError err = new StandardError(Instant.now(), status.value(), error, e.getMessage(), request.getRequestURI());
        return ResponseEntity.status(status).body(err);
    }
}
```

Essa classe irá fazer as comunicações entre as mensagens de erro e a sua manipulação com o StandardError.

## Correções nos Services

Após isso, iremos fazer várias correções das chamadas ao repositório. Dentre elas estão:

### delete
```
public void delete(Long id){
        try{
            repository.deleteById(id);
        } catch (EmptyResultDataAccessException e){
            throw new NotFoundException(id);
        } catch (DataIntegrityViolationException e){
            throw new DatabaseException(e.getMessage());
        }
    }
```

### update
```
public Class update(Long id, Products obj){
        try{
            Class entity = repository.getReferenceById(id);
            updateData(entity, obj);
            return repository.save(entity);
        } catch (EntityNotFoundException e){
            throw new NotFoundException(id);
        }
    }
```

### findById (correções)
```
public Class findById(Long id){
    Optional<Class> obj = repository.findById(id);
    return obj.orElseThrow(()-> new NotFoundException(id));
    }
```

Neste último, observe que o obj pode retornar, além do get, outras funções.

---

## Paginação

Para que a paginação ocorra de forma correta, devemos fazer algumas modificações no nosso código. Uma delas será o retorno no nosso FindAll. Ao invés de retornamos uma list, iremos retorna uma Page. Além disso, nós iremos por o seguinte código na nossa requisição. Mas, antes de colocar, vejamos os seus parâmetros:

```
@RequestParam(value = "page", defaultValue = "0") Integer page,
// por padrão, o valor default é 0. É por onde iremos começar, se é a página 0 ou outra.
@RequestParam(value = "linesPerPage", defaultValue = "12") Integer linesPerPage,
// Aqui é o valor de linhas por página.
@RequestParam(value = "direction", defaultValue = "DESC") String direction,
// Direção: Asc ou Desc.
@RequestParam(value = "orderBy", defaultValue = "name") String orderBy
// Nome do atributo que nós vamos ordenar os registros. No exemplo dos usuários, retornaremos os nomes.
```

Eles ficarão dentro dos parênteses do findAll. Além disso, iremos implementa mais coisas no corpo do nosso método.
```
public ResponseEntity<Page<User>> findAll(
    @RequestParam(value = "page", defaultValue = "0") Integer page,
    @RequestParam(value = "linesPerPage", defaultValue = "12") Integer linesPerPage,
    @RequestParam(value = "direction", defaultValue = "DESC") String direction,
    @RequestParam(value = "orderBy", defaultValue = "name") String orderBy
){

        PageRequest pageRequest = PageRequest.of(page, linesPerPage, Direction.valueOf(direction), orderBy);
        // Os valores precisam ser, necesessariamente nessa ordem.

        Page<User> list = service.findAllPaged(pageRequest);

        return ResponseEntity.ok().body(list);
    }
```

**Service**

```
@Transactional
public Page<UserDTO> findAllPaged(PageRequest pageRequest){
    Page<User> list = repository.findAll(pageRequest);
	return list.map(x -> new UserDTO(x));
}
```

Ao fazer a requisição no Postman, nós podemos fazer a requisição de outras páginas da seguinte forma:
```
http://localhost/users?page=1
```

Esse "?" serme para colocar parâmetros opcionais. Além disso, podemos trocar os outros parâmetros.
```
http://localhost/users?page=1&linesPerPage=5&direction=ASC&orderBy=name
```

---

## Banco de dados NoSQL

### Conectando o mongodb

Utilizando o STS (que foi a versão que achei melhor para fazer isso), em **application.properties**, nós iremos adicionar a seguinte linha:
```
spring.data.mongodb.uri=mongodb://localhost:27017/udemy
```

Além disso, nós devemos importar as dependencias maven no mongodb stater:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

Nela irá o endereço, a porta e o nome da coleção.

Após isso, devemos iniciar o mongo com o seguinte comando no terminal:
```
mongod
```

Obs.: Não poderemos de esquecer de criar a coleção que será usada. Podemos usar o MongoDB Compass para criá-la.

### Criação

A criação dos arquivos segue o mesmo exemplo de como seria criar em um bando SQL. Há algumas diferenças. Entre elas estão a mudança de nome. O que nós conhecemos como **@Entity**, no NoSQL será **@Document**.

Além disso, no Repository, ao invés do **JpaRepository**, nós iremos usar o **MongoRepository**. Da mesma forma, o tipo de id será uma String.

### Interações com o banco de dados

#### Obtendo um usuário por id

Para fins de tratamento de exceção, devemos criar uma classe ObjectNotFoundException no subpacote (o qual criaremos) **service.exception**.

Esse será o código que haverá dentro da classe:

```
public class ObjectNotFoundException extends RuntimeException {

	private static final long serialVersionUID = 1L;
	
	public ObjectNotFoundException(String msg) {
		super(msg);
	}	
}
```

Além disso, podemos seguir os passor que tem no [Tratamento de exceções](#tratamento-de-excecoes) no subpacote **resources.exception**. Nós iremos criar o StandardError e o ResourceExceptionHandler.


**Service**

Iremos adicionar o método findById.

```
public User findById(String id){
    Optional<User> obj = repository.findById(id);
	return obj.orElseThrow(() -> new ObjectNotFoundException("Objeto não encontrado"));
}
```

**Resource ou Controller**
```
@GetMapping("/{id}")
public ResponseEntity<UserDTO> findById(@PathVariable String id){
    User obj = service.findById(id);
    return ResponseEntity.ok().body(new UserDTO(obj));
}
```