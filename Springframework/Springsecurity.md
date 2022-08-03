# Springsecurity

O srping security trabalha com a segurança da nossa API. Através dela nós podemos criptografar senhas, dar autorização a alguns usuários e assim deixá-lo mais segura.

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