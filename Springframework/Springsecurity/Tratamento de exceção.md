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
