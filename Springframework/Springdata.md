# Springdata
Usado como implementação ao JPA trabalhando, principalmente, na percistencia de dados.

## Instalação
Para iniciar o projeto, nós iremos no site do [spring initializr](https://start.spring.io/). Mas, dessa vez, nós iremos implementar a dependência **Spring Data JPA** e do Banco de dados. No meu caso, **MySQL Driver**.

## JpaRepository
Quando vamos criar uma classe para ser considerada uma tabela, fazemos normalmente de acordo coma o mapeamento do Jpa. O SprintgData vai atuar na implementação dessas datas.

1. Criemos uma interface que irá extender JpaRepository. Nele será indicado qual será a tabela que criamos (a classe que criamos para isso) e o tipo de ID.
```
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

Além disso, devemos adicionar a **dependência maven** do nosso banco de dados. Neste caso, MySQL.
