## Algumas anotations

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
- [@ManyToOne](#manytoone)
- [@OneToMany](#onetomany)
- [@ManyToMany](#manytomany)
- [@JsonIgnore](#jsonignore)
- [@Column](#column)
- [@Table](#table)

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
public User findById(Long id){
    Optional<User> obj = repository.findById(id);
    User entity = obj.orElseThrow(()-> new NotFoundException(id));
    return new UserDTO(entity);
}
```

**Controller**:

```
@GetMapping("/{id}")

public ResponseEntity<UserDTO> findById(@PathVariable Long id)  { //ResponseEntity pertence ao spring
    UserDTO obj = service.findById(id);
    reuturn ResponseEntity.ok().body(obj); // Para a requisição ser exposta no body.
} 
```

#### FindAll

Obs.: Recomenda-se o modelo paginado.

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
@Transactional
public UserDTO insert(UserDTO dto){
    User entity = new User();
    entity.setEmail(dto.getEmail());
    entity.setRole(dto.getRole());
    ...
    entity = repository.save(entity);
    return new UserDTO(entity);
	}

...

private CopyDtoToEntity(UserDTO dto, User entity){
    entity.setName(dto.getName());
    entity.setDescription(dto.getDescription());
    ...

    // Essa parte pertence à categoria. Aqui é uma lista.
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
public ResponseEntity<UserDTO> insert(@RequestBody UserDTO obj){
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

    // Essa parte pertence à categoria. Aqui é uma lista.
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

Em alguns momentos nós poderemos desejar que a classe Product, ao ser carregada, leve consigo a classe category, a qual fará essa ligação ManyToMany. Então, para isso, devemos instanciar o FetchType EAGER. Para que o efeito seja o contrário, utilizamos o LAZY ao invés do EAGER.

```
public class Product ...
...
@ManyToMany(fecth = FetchType.EAGER)
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

Caso queiramos que os valores vão com o Timezone local, **substituímos** o without por with.

```
@Column(columnDefinition = "TIMESTAMP WITHOUT TIME ZONE")
private Instante date;
```

Além dela, outra forma de trabalhar é com os textos. Nós definimos o tipo de coluna para "text" para que sejam aceitos mais do que 255 caracteres. Geralmente é utilizado para descrições.
```
@Column(columnDefinition = "TEXT")
private String description;
```

Temos também a propriedade Unique que faz um certo atributo ser único.
```
@Column(unique = true)
private String email;
```

### @Table

Utilizado, geralmente, para mudar o nome da entidade a fim de que ela não seja o mesmo da Classe Java.
```
@Table(name = "studentData")
public class Student{
    ...
}
```