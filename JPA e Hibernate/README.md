# Aula x

## Intalação no Maven através do Intellij

**Maven do JPA**
```
<!-- https://mvnrepository.com/artifact/javax.persistence/javax.persistence-api -->
<dependency>
    <groupId>javax.persistence</groupId>
    <artifactId>javax.persistence-api</artifactId>
    <version>2.2</version>
</dependency>
```

**Maven do JDBC**
```
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.29</version>
</dependency>
```

**Maven do Hibernate**
```
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-core -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.12.Final</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-entitymanager -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>5.4.12.Final</version>
</dependency>
```

## Configuração para o projeto

### 1. Criar pasta META-INF em resoucers e, em seguida, criar o arquivo persistence.xml
![Imagem mostrando a pasta META-INF e o arquivo persistence.xml](./Foto/metainf%20e%20pesristence.png)

### 2. Código para por no arquivo
```
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
version="2.1">
    <persistence-unit name="<nome-do-persist>" transaction-type="RESOURCE_LOCAL">
        <properties>
            <property name="javax.persistence.jdbc.url"
            value="jdbc:mysql://localhost/<nome_do_banco_de_dados>?useSSL=false&amp;serverTimezone=UTC" />

            <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />

            <property name="javax.persistence.jdbc.user" value="<usuario_do_mysql>" />

            <property name="javax.persistence.jdbc.password" value="<senha_do_mysql>" />

            <property name="hibernate.hbm2ddl.auto" value="update"/>


            <!-- Caso a parte de baixo dê errado, pode-se substituri o dialect Mysql 8 por outra no site do link abaixo -->
            <!-- https://docs.jboss.org/hibernate/orm/5.4/javadocs/org/hibernate/dialect/package-summary.html -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL8Dialect" />
        </properties>
    </persistence-unit>
</persistence>
```

---

## Configuração da classe que irá para a tabela
Nesse processo, nós criamos uma classe. Essa classe que criamos irá subir para o nosso banco de dados.
Quando ela for completamente criada, com os seus devidos atributos, nós iremos precisar implementar alguns elementos. Mas antes disso, não esqueça de implementar o Serializable e a constante serialVersionUID:
```
public class Nome implements Serializable {
    private static final long serialVersionUID = 1L;
    ...
}
```

### @Entity
Para indicar que a mapeação da tabela será dessa classe que criamos.
```
@Entity
public class Nome implements Serializable {...}
```

### @Id e @GeneratedValue(strategy= GenerationType.IDENTITY)
Para mapear o ID e implementar o id de forma automática.
```
@Id
@GeneratedValue(strategy= GenerationType.IDENTITY)
private Integer id;
```

## @Colum
Além disso, nós também podemos definir o nome que estará em alguma coluna caso quisermos alterar. Como valor padrão, ele vem o nome da propriedade que colocamos.
```
@Id
@GeneratedValue(strategy= GenerationType.IDENTITY)
@Colum(name = "id_name")
private Integer id;
```

---

## EntityManeger
Um EntityManager mapea um conjunto de classes a um banco de dados particular. Este conjunto de classes é chamado de persistence unit (unidade de persistência).
```
EntityManager em = emf.createEntityManager();

```

## EntityManagerFactory
EntityManagers podem ser criado ou podem ser obtidos de um EntityManagerFactory. Em uma aplicação Java SE, você tem que usar um EntityManagerFactory para criar instancias de um EntityManager. Usar o factory não é uma exigência em um ambiente Java EE.
```
EntityManagerFactory emf = Persistence.createEntityManagerFactory("exemplo-jpa");
```

### Adicionando no Bando de Dados
```
Pessoa p1 = new Pessoa(null, "Carlos", "carlos@gmail.com");
Pessoa p2 = new Pessoa(null, "Joaquin", "joaquin@gmail.com");
Pessoa p3 = new Pessoa(null, "Maria", "maria@gmail.com");

EntityManagerFactory emf = Persistence.createEntityManagerFactory("exemplo-jpa");
EntityManager em = emf.createEntityManager();

// Ao iniciar alguma mudança no banco de dados, devemos iniciar o begin para permitir as alterações.
em.getTransaction().begin();

// O método persist salva os elementos no banco de dados
em.persist(p1);
em.persist(p2);
em.persist(p3);

// Sempre que fazemos alguma alteração e desejamos enviar, precisamos confirmar através desse método:
em.getTransaction().commit();

System.out.println("Fim");
// Finalizar conexão
em.close();
emf.close();
```

### Encontrando e removendo no Banco de Dados
```
EntityManagerFactory emf = Persistence.createEntityManagerFactory("exemplo-jpa");
EntityManager em = emf.createEntityManager();

// Para localizar algum dado com base no ID
Pessoa p = em.find(Pessoa.class, 2);

em.getTransaction().begin();
// O método remove irá deletar o dado procurado, o qual armazenamos em p
em.remove(p);

em.getTransaction().commit();

System.out.println("Fim");
em.close();
emf.close();
```