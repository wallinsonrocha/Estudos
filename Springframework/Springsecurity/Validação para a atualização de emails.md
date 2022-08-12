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
