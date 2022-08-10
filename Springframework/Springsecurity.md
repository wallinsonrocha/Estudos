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

# A partir daqui, será feito a configuração do Authorization server, que enviar as credencias através de um token para o usuário.

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
    // O role é uma entidade criada.

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

---

# Autorização e autenticação - OAuth2

## Configuração para externalizar alguns valores

Nós usaremos o arquivo application.properties para criar as variáveis. Adicionaremos as seguintes linhas:

```
spring.profiles.active=${APP_PROFILE:test}

spring.jpa.open-in-view=false

security.oauth2.client.client-id=${CLIENT_ID:dscatalog}
security.oauth2.client.client-secret=${CLIENT_SECRET:dscatalog123}

jwt.secret=${JWT_SECRET:MY-JWT-SECRET}
jwt.duration=${JWT_DURATION:86400}
```

## Beans para Token JWT

Pode ser colocado no AppConfig. Eles serão capazes de acessar o token.

```
@Value("${jwt.secret}")
    private String jwtSecret;

@Bean
public JwtAccessTokenConverter accessTokenConverter() {
	JwtAccessTokenConverter tokenConverter = new JwtAccessTokenConverter();
	tokenConverter.setSigningKey(jwtSecret);
	return tokenConverter;
}

@Bean
public JwtTokenStore tokenStore() {
	return new JwtTokenStore(accessTokenConverter());
}
```

## Classe

Nós iremos criar ela no pacote de configuração.

```
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter{

    @Value("${security.oauth2.client.client-id}")
    private String clientId;

    @Value("${security.oauth2.client.client-secret}")
    private String clientSecret;

    @Value("${jwt.duration}")
    private Integer jwtDuration;

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Autowired
    private JwtAccessTokenConverter accessTokenConverter;

    @Autowired
    private JwtTokenStore tokenStore;

    // O bean criado no SecurityConfig
    @Autowired
    private AuthenticationManager authenticationManager;

    // Sobscreveremos os 3 métodos Override configure
    @Override
    protected void configure(AuthorizationServerSecurityConfigurer security) throws Exception{
        security.tokenKeyAccess("permitAll()").checkTokenAccess("isAuthenticated()");
    }

    @Override
    protected void configure(ClientDetailsServiceConfigurer clients) throws Exception{
        clients.inMemory()
        .withClient(clientId)
        .secret(passwordEncoder.encode(clientSecret))
        .scopes("read", "write")
        .authorizedGrantTypes("password")
        .accessTokenValiditySeconds(jwtDuration);
    }

    @Override
    protected void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception{
        endpoints.authenticationManager(authenticationManager)
        .tokenStore(tokenStore)
        .accessTokenConverter(accessTokenConverter);
    }
}
```

---

# Autorização e autenticação - Acesso na url

Quando vamos fazer o acesso, o endpoint será "/oauth/token". Além disso, no postman, iremos colocar o método post, pois faremos um loggin. Ao invés de fazermos pela aba do "Params", iremos na aba "Authorization". E o tipo de autorização que escolheremos é o basic.

![Direcionamento segundo a instrução acima](./Fotos/Captura%20de%20tela%202022-08-06%20101622.png)

Logo após isso, iremos colocar o Usuário e a senha de acordo com o que colocamos no nosso banco. No caso do código acima foram "usuario" e "senha123".

Após isso, precisaremos colocar as credenciais do usuário. Iremos na aba Body e escolheremos o formato x-www-form-urlencoded.

![Direcionamento segundo a instrução acima](./Fotos/credenciais%20do%20usuario.png)

Logo após, iremos colocar as chaves e os valores para as credenciais. O usuário e a senha devem estar registradas no banco de dados.

![Direcionamento segundo a instrução acima](./Fotos/chaves%20e%20valores.png)

---

# Adicionando informações no Token

Caso a nossa aplicação precise de algumas informações, como o nome, id, email ou outras coisas nescessárias de alguém que foi autenticado, podemos fazer isso da seguinte forma:

Criaremos um pacote chamado **Components** para por os nossos componentes.

**Classe JwtTokenEnhancer**

```
@Component
public class JwtTokenEnhancer implements TokenEnhancer{
    
    // Esse repositório irá auxiliar o método abaixo
    @Autowired
    private UserRepository userRepository;

    @Override
    // Método OAuth2AccessToken exigido após implantar o TokenEnhancer (accessToken, authentication) {
        User user = userRepository.findByEmail(authentication.getName());

        // Adicionando objetos no token. Eles estarão dentro do token
        Map<String, Object> map = new HasMap<>();
        map.put("userFirstName", user.getFisrtName());
        map.put("userId", user.getId());

        // Consolidando os objetos adicionados.
        // Esse Default é bem mais específico do que o OAuth2AccessToken. Ele possui o método para adicionar.
        DefaultOAuth2AccessToken token = (DefaultOAuth2AccessToken) accessToken;
        token.setAdditionalInformation(map);

        return accessToken; // Ou token.
    }
}
```

**Injetando na classe** no AuthorizationServerConfig.

```
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter{
    ...
    @Autowired
    private JwtTokenEnhancer tokenEnhancer;
    ...

     @Override
    protected void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception{

        TokenEnhancerChain chain = new TokenEnhancerChain();
        // Esse objeto espera uma lista
        chain.setTokenEnhancer(Arrays.asList(accessTokenConverter, tokenEnhancer));

        endpoints.authenticationManager(authenticationManager)
        .tokenStore(tokenStore)
        .accessTokenConverter(accessTokenConverter)
        //Adicional
        .tokenEnhancer(chain);
}
```

# A partir daqui, será feito a configuração do Resource server, que enviará alguns resources caso o usuário esteja autorizado através de um token. Esse passo só poderá ser feito caso o anterior tenha sido feito completamente.

# Resource

Dentro do pacote config criaremos a seguinte classe:

**ResourceServerConfig**

```
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter{

    @Autowired
    private JwtTokenStore tokenStore;

        // Para organizar a autorização de resource para usuário
        // Aqui estão as rotas que todos podem acessar
        private static final String[] PUBLIC = { "/oauth/token" };

        // Aqui estão as rotas que somente o operador e o admin podem acessar.
        // O "**" serve para indicar "todos".
        private static final String[] OPERATOR_OR_ADMIN = { "/products/**", "/categories/**" };

        // Resources para o Admin
        private static final String[] ADMIN = { "/users/**" };



    // Iremos implementar 2 Override. O Configure HttpSecurity e o ResourceServerSecurityConfigurer.
    // Não podermos esquecer que ele está no Source (Alt+Shift+S no STS).

    @Override
    // (ResourceServerSecurityConfigurer resources){
        resources.tokenStore(tokenStore);
    }

    @Override
    // (HttpSecurity http) {
        // Configuração de quem pode acessar tal resource.
        // É através dele que fazemos essas configurações. Baste procurar o método requerido e implementar.
        http.authorizedRequests()
        .antMatchers(PUBLIC).permitAll() // Autorizações
        .antMatchers(HttpMethod.GET, OPERATOR_OR_ADMIN).permitAll() // Apenas os métodos get nessas rostas
        .antMatchers(OPERATOR_OR_ADMIN).hasAnyRoles("OPERATOR", "ADMIN")
        .antMatchers(ADMIN).hasAnyRoles("ADMIN")
        .anyRequest().authenticated(); // Exige que, qualquer outra rota, o usuário esteja logado.
    }
}
```

## Configuração para o H2 Console

Para que seja permitido entrar no H2 console como faziamos antes, devemos configurar sua permissão. Para isso, nas rotas públicas devemos colocar o seu resource lá.

```
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter{
    ...
    @Autowired
    private Environment env;

    private static final String[] PUBLIC = { "/oauth/token", "/h2-console/**" };
    ...

    @Override
    // (HttpSecurity http) {

        // Configuração para liberar o H2
        if(Arrays.asList(env.getActiveProfiles()).contains("test")){
            http.headers().frameOptions().disable();
        }

        http.authorizedRequests()
        .antMatchers(PUBLIC).permitAll() 
        .antMatchers(HttpMethod.GET, OPERATOR_OR_ADMIN).permitAll()
        .antMatchers(OPERATOR_OR_ADMIN).hasAnyRoles("OPERATOR", "ADMIN")
        .antMatchers(ADMIN).hasAnyRoles("ADMIN")
        .anyRequest().authenticated();
    }
}
```

## Para o postman na aba Tests

```
if(responseCode.code >= 200 && responseCode.code >= 300){
    var json = JSON.parse(responseBody);
    postman.setEnvironmentVariable('token', json.access_token);
}
```