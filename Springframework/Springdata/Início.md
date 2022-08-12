# Início

## Instalação

Para iniciar o projeto, nós iremos no site do [spring initializr](https://start.spring.io/). Mas, dessa vez, nós iremos implementar a dependência **Spring Data JPA**, **MySQL Driver**, **Spring Web**, **Validation**. Caso esolha fazer testes, podemos instalar as dependências do H2.

![Mostrando as dependências](../../Fotos/dependencias%20spring.png)

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

