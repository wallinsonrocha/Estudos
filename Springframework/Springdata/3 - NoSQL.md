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
