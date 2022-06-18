# Springdata
Usado como implementação ao JPA trabalhando, principalmente, na percistencia de dados.

## Instalação
Para iniciar o projeto, nós iremos no site do [spring initializr](https://start.spring.io/). Mas, dessa vez, nós iremos implementar a dependência **Spring Data JPA** e do Banco de dados. No meu caso, **MySQL Driver**.

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

## Conectando com o MySQL.
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

Além disso, devemos adicionar a **dependência maven** do nosso banco de dados. Neste caso, MySQL.

---

## Algumas anotations 
### @RestController
Serve para identificar a classe como Bean do tipo controller. Geralmente é colocado para gerar as dependências quando necessário. É colocado na classe principal da nossa aplicação. Dentro da classe que recebe essa anotation é colocada os GetMapping. Ela irá fazer as chamadas no service que irá fazer a comunicação com o repository.

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
RI a nível de classe. Serve para identificar por onde nós podemos acessar o recurso criado. Quando há algum método, seja ele get, post... e não houver alguma direção para ele, eles serão encaminhados para cá e dará prosseguimento ao processo.
```
@RestController
@CrossOrigin(origin = "*", maxAge = 3600)
@RequestMapping("/direcao")
public class MyAppController{
    ...
}
```

@RestController
@CrossOrigin(origin = "*", maxAge = 3600)
@RequestMapping("/direcao")
public class MyAppController{
    ...
}
```
```

### @GetMapping()
Serve para identificar tal método através do direcionamento da url. Caso seu objetivo seja ele ir para o RequestMapping, não precisamos identificar a sua direção.
```
@GetMapping("/direcao")
public String index(){
    return "Hello World";
}
```

### PostMapping()
Serve para identificar tal método como post. Caso seu objetivo seja ele ir para o RequestMapping, não precisamos identificar a sua direção.
Exemplo:
```
public ResponseEntity<Object> saveEstacionamento(@RequestBody @Valid EstacionamentoDto estacionamentoDto){
    var estacionamentoModel = new EstacionamentoModel();
    BeansUtils.copyProperties(estacionamentoDto, estacionamentoModel); // Conversão de ? para ?.
    estacionamentoModel.setRegistrationDate(LocalDateTime.now(ZoneId.of("UTC"))); // Set da data de registro
    return ResponseEntity.status(HttpStatus.CREATED.body(estacionamentoService.save(estacionamentoModel))); 
    // O save e o estacionamentoService devem ser criados na classe que possui o @Service
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

---

## Anotations do Spring Validation
Elas vão, em sua maioria, em atributos. Servem para verificá-las.

### NotBlack
Para verificar se o valor não está vazio.

### Size()
Para verificar se o campo tem mais ou menos caracteres. Para isso, identificamos com "max = numero" ou "min = numero".