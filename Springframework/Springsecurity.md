# Springsecurity

O srping security trabalha com a segurança da nossa API. Através dela nós podemos criptografar senhas, dar autorização a alguns usuários e assim deixá-lo mais segura.

---

##  SecurityConfig - Autorização para usar a API.

Quando nós criamos alguma aplicação com springsecurity, normalmente ele gera um código que deve ser implementado na requisição de uma aplicação para que o se usos seja altorizado. Mas, podemos configurar para que o código rode sem precisar dessa autorização. Para isso, devemos criar uma **classe de configuração** no pacote **config**.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{
    @Override
    public void configure(WebSecurity web) throws Exception{
        web.ignoring().antMatchers("/**");
    }
}
```

---

## Beans Validations

Com o beans validations nós podemos verificar a validação de cada atributo da nossa API. Mas, para isso, nós devemos implementar as suas dependências.

Para que todas essas anotações abaixo possam funcionar, nós devemos, também, colocar a anotation **@Valid** no nosso Controller:
```
@PostMapping
public ResponseEntity<ProductDTO> insert(@Valid @RequestBody ProductDTO dto){
    ...
}
```

### @NotEmpty

Para obrigar o preenchimento de algum atributo.

```
@NotEmpty(message = "Campo obrigatório")
private String firstName;
```

### @NotBlank

Para obrigar o preenchimento do campo, mas que não seja apenas espaços.
```
@NotEmpty(message = "Campo obrigatório")
private String lastName;
```

### @Email

Para a validação de email. 

```
@Email(message = "Favor, entrar com um email válido")
private String email;
```

### @Positive

Obriga algum atributo number a ser positivo.

```
@Positive(message = "O preço deve ser positivo")
private String price;
```

### @PastOrPresent

Uma anotação que faz com que a data inserida seja passada ou presente.

```
@PastOrPresent(message = "A data não pode ser futura")
private Instant date;
```

### @PastOrPresent

Uma anotação que faz com que a data inserida seja futura ou presente.

```
@FutureOrPresent(message = "A data não pode ser passada")
private Instant date;
```

### @Size

Define o tamanho permitido em alguma string.

```
@Size(min = 5, max = 60, message = "Deve ter entre 5 a 60 caracteres)
@NotEmpty(message = "Campo obrigatório")
private String lastName;
```

---

## BCryptPasswordEncoder - API para criptografia.

Essa API é obtida quando colocamos a dependência do springsecurity na nossa aplicação.

Para prepararmos a API para isso, precisamos criar uma classe de configuração. Então, após criar um pacote **config**, nós colocaremos o seguinte código na classe de configuração:

```
@Configuration
public class AppConfig{

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

Ela será utilizada na criação de senhas quando se cria um novo usuário ou atualiza algum. Nós colocaremos assim:

```
@Autowired
public BCryptPasswordEncoder passwordEncoder;

@Transactional
public UserDTO insert(UserInsertDTO dto){
    User entity = new User();
    copyDtoToEntity(dto, entity);
    entity.setPassword(passwordEnconder.encode(dto.getPassword()));
    entity = repository.save(entity);
    return new UserDTO(entity);
}
```

Obs.: Esse **UserInsertDTO** é uma classe que extend **UserDTO** porém tem mais um atributo, que é a senha.

---

## Tratamento de exceção - MethodArgumentNotValidException

Esse erro é disparado quando a validação de algum atributo está errado.

Mesmo que capturemos esse erro, ainda precisamos filtrar mais ainda para coletar a mensagem de erro principal na nossa API. Para isso, nós precisaremos criar uma classe **ValidationError** que irá extender a nossa classe **StandardError**. Lá nós iremos criar um atributo que irá ter uma lista de objetos da Classe **FieldMessage**, que criaremos.

**FieldMessage**

```
public class FieldMessage implements Serializable {
    private static final long serialVersionUID = 1L;

    private String field;
    private String message;

    ...construtor
    ...getters e setters
}
```

**ValidationError**

```
public class ValidationError extends StandardError{
    private List<FieldMeassge> errors = new ArrayList<>();

    public List<FieldMessage> getErrors(){
        return errors;
    }

    public void addError(String fieldName, String message){
        errors.add(new FieldMessage(fieldName, message));
    }
}
```

**Modificações na classe ResourceExceptionHandler**

Na classe Handler, a mesma que fora criada no springdata, nós a implementamos para personalizá-la.

```
// Isso indica que essa classe irá tratar erros especiais.
@ControllerAdvice
public class ResourceExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationError> validation(MethodArgumentNotValidException e, HttpServletRequest request){
        String error = "Validation error";
        HttpStatus status = HttpStatus.UNPROCESSABLE_ENTITY;
        ValidationError err = new ValidationError(Instant.now(), status.value(), error, e.getMessage(), request.getRequestURI());

        // FieldError pertence à biblioteca Beans Validation
        for (FieldError f : e.getBindingResult().getFieldErrors()){
            err.addError(f.getField(), f.getDefaultMessage());
        }

        return ResponseEntity.status(status).body(err);
    }
}
```

---

## Validação para a criação de um email

Para isso, faz-se necessário criarmos algumas classes para esse propósito.

### ConstraintValidator UserInsertValid

É um formato padrão. Ele irá se repetir em todo projeto que criarmos. Essa classe é utilizada para a criação de anotations. Com ele iremos custominzar a nossa. Nós iremos por ela no pacote de **serviços**. O nome será **UserInsertValid**.

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

@Constraint(validatedBy = UserInsertValidator.class)
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)

public @interface UserInsertValid {
	String message() default "Validation error";

	Class<?>[] groups() default {};

	Class<? extends Payload>[] payload() default {};
}
```

### UserInsertValidator

Para validar a anotation, nós deveremos criar uma classe com o nome de **UserInsertValidator**, que tem como referência no @Constraint do código acima.

```
import java.util.ArrayList;
import java.util.List;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.springframework.beans.factory.annotation.Autowired;

import com.devsuperior.dscatalog.dto.UserInsertDTO;
import com.devsuperior.dscatalog.entities.User;
import com.devsuperior.dscatalog.repositories.UserRepository;
import com.devsuperior.dscatalog.resources.exceptions.FieldMessage;

public class UserInsertValidator implements ConstraintValidator<UserInsertValid, UserInsertDTO> {
	
	@Override
	public void initialize(UserInsertValid ann) {
	}

	@Override
	public boolean isValid(UserInsertDTO dto, ConstraintValidatorContext context) {
		
		List<FieldMessage> list = new ArrayList<>();
		
		// Coloque aqui seus testes de validação, acrescentando objetos FieldMessage à lista
		
		for (FieldMessage e : list) {
			context.disableDefaultConstraintViolation();
			context.buildConstraintViolationWithTemplate(e.getMessage()).addPropertyNode(e.getFieldName())
					.addConstraintViolation();
		}
		return list.isEmpty();
	}
}
```

### Adicionando método no UserRepository

Para que ele possa executar os testes, nós devemos examinar o banco de dados. Para isso, usaremos o repository do User. Porém, no nosso repositório não um método que chame os usuários por email. Então precisaremos criar. Na interface **UserRepository**, acrescentaremos:

```
User findByEmail(String email);
```

### Código completo do UserInsertValidator usando o repository

```
import java.util.ArrayList;
import java.util.List;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.springframework.beans.factory.annotation.Autowired;

import com.devsuperior.dscatalog.dto.UserInsertDTO;
import com.devsuperior.dscatalog.entities.User;
import com.devsuperior.dscatalog.repositories.UserRepository;
import com.devsuperior.dscatalog.resources.exceptions.FieldMessage;

public class UserInsertValidator implements ConstraintValidator<UserInsertValid, UserInsertDTO> {

    @Autowired
    private UserRepository repository;
	
	@Override
	public void initialize(UserInsertValid ann) {
	}

	@Override
	public boolean isValid(UserInsertDTO dto, ConstraintValidatorContext context) {
		
		List<FieldMessage> list = new ArrayList<>();
		
		User user = repository.findByEmail(dto.getEmail);

        if(user != null){
            list.add(new FieldMessage("email", "Email já existe"));
        }
		
		for (FieldMessage e : list) {
			context.disableDefaultConstraintViolation();
			context.buildConstraintViolationWithTemplate(e.getMessage()).addPropertyNode(e.getFieldName())
					.addConstraintViolation();
		}
		return list.isEmpty();
	}
}
```

### Adicionando a anotation no UserInsertDTO

Logo após, assim que terminarmos, nós iremos precisar colocar essa anotação criada no UserInsertDTO.

```
@UserInsertValid
public class UserInsertDTO extends UserDTO {
    private static final long serialVersionUID = 1L;
    ...
}
```

---

## Validação para a atualização de um email

Para isso, faz-se necessário criarmos algumas classes para esse propósito.

### ConstraintValidator UserUpdateValid

É um formato padrão. Ele irá se repetir em todo projeto que criarmos. Essa classe é utilizada para a criação de anotations. Com ele iremos custominzar a nossa. Nós iremos por ela no pacote de **serviços**. O nome será **UserUpdateValid**.

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

@Constraint(validatedBy = UserUpdateValidator.class)
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)

public @interface UserUpdateValid {
	String message() default "Validation error";

	Class<?>[] groups() default {};

	Class<? extends Payload>[] payload() default {};
}
```

### UserUpdateValidator

Para validar a anotation, nós deveremos criar uma classe com o nome de **UserUpdateValidator**, que tem como referência no @Constraint do código acima. Não podemos esquecer de dar **ctrl+shift+o** para as importações automáticas.

```
import java.util.ArrayList;
import java.util.List;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import org.springframework.beans.factory.annotation.Autowired;

import com.devsuperior.dscatalog.dto.UserInsertDTO;
import com.devsuperior.dscatalog.entities.User;
import com.devsuperior.dscatalog.repositories.UserRepository;
import com.devsuperior.dscatalog.resources.exceptions.FieldMessage;

public class UserUpdateValidator implements ConstraintValidator<UserUpdateValid, UserUpdateDTO> {

    // Para acessar o ID da requisição
    @Autowired
    private HttpServletRequest request;

    @Autowired
    private UserRepository repository;
	
	@Override
	public void initialize(UserUpdateValid ann) {
	}

	@Override
	public boolean isValid(UserUpdateDTO dto, ConstraintValidatorContext context) {

        // Para ignorar o aviso
        @SuppressWarnings("unchecked")
        // Para fazer o acesso. Além disso, deve haver um down cast para string, já que trabalhamos com http
        var uriVars = (Map<String, String>) request.getAttribute(HandlerMapping.URI_TEMPLATE_VARIABLE_ATTRIBUTE);
        long userId = Long.parseLong(uriVars.get("id"));
		
		List<FieldMessage> list = new ArrayList<>();
		
		User user = repository.findByEmail(dto.getEmail);

        if(user != null && userId != user.getId()){
            list.add(new FieldMessage("email", "Email já existe"));
        }
		
		for (FieldMessage e : list) {
			context.disableDefaultConstraintViolation();
			context.buildConstraintViolationWithTemplate(e.getMessage()).addPropertyNode(e.getFieldName())
					.addConstraintViolation();
		}
		return list.isEmpty();
	}
}
```

Para podermos usar a anotation, não podemos colocá-lo no UserDTO. Devereremos criar uma outra classe chamada **UserUpdateDTO**. Dentro dele haverá:

```
@UserUpdateValid
public class UserUpdateDTO extends UserDTO {
    private static final long serialVersionUID = 1L;
}
```

### Modificação no Update

Além disso, precisaremos modificar o método Update em UserResource e no UserService. 

**Controller**

Ao invés dele receber UserDTO, ele irá receber UserUpdateDTO.

```
@PutMapping(value = "/{id}")
public ResponseEntity<User> update(@PathVariable Long id, @RequestBody UserUpdateDTO obj){
    UserDTO newDto = service.update(id, obj);
    return ResponseEntity.ok().body(newDto);
}
```

**Service**

```
public UserDTO update(Long id, UserUpdateDTO dto){
    try{
        User entity = new User();
        
        CopyDtoToEntity(dto, entity);

        entity = respository.save(entity);
        return new UserDTO(entity);
    } catch(EntityNotFoundException e) {
        throw new ResourceNotFoundException("Id not found " + id);
    }
}
```

---

# Autorização e autenticação

Dependências: **spring security test e spring security oauth2 autoconfigure**

Para poder fazer a comunicação com o servidor e conseguir um token de acesso.

## Interfaces

### UserDetails

É uma interface que será implementada na entidate do Usuário. Quando implementarmos, ele irá criar os métodos que faltam.

```
public class User implements UserDetails, Serializable{
    ...

    @Override
    public String getUsername(){
        return email;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities(){
        return roles.stream().map(role -> new SimpleGrantedAuthority(role.getAuthority())).collect(Collectors.toList());
    }

    // O role aqui é o papel de cada usuário de acordo com o exemplo visto na video aula.

    ...
}
```

De restante dos métodos, a fins de testes, podemos retornar true para todos.

### UserDetailsService

É uma interface que será implementada no UserService.

```
public class UserService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Já criado no nosso repositório
        User user = repository.findByEmail(username);
        if(user == null){
            logger.error("User not found: " + username);
            throw new UsernameNotFoundException("Email not found");
        }

        logger.info("User found: " + username);
        return user;
    }
}
```

**Implementação de um Logger**

```
private static Logger logger = LoggerFactory.getLogger(UserService.class);
```

## Classes

É o mesmo SecurityConfig que criamos, porém, podemos renomear para WebSecurityConfig se quisermos. Iremos fazer algumas alterações.

```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter{

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public void configure(WebSecurity web) throws Exception{
        web.ignoring().antMatchers("/actuator/**");
    }

    // Precisamos criar uma sobrecarga para o mesmo método configure. Nas configurações de geração, como o construtor e getters e setters, tem o de override. Iremos escolher o configure(AuthenticationManagerBuilder). Nele iremos usar as configurações como BCrypt e o UserDetailsSerice.
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder);
    }

    // Chamaremos mais um Override
    @Override
    @Bean
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }
}
```