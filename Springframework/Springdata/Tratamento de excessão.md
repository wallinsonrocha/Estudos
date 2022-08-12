## Tratamento de exceções

Ao criar as funções do CRUD, em alguns momentos poderá haver um erro ao requisitar um get, push ou qualquer outra coisa da nossa API. Por conta disso, precisamos fazer um tratamento para isso.

### Início

Para iniciar, devemos, dentro da pasta Exception, criar alguns arquivos e uma pasta. Os arquivos são:
1. DatabaseException - que irá extender Runtime Exception.
2. NotFoundException - que também irá extender de Runtime Exception.
3. Resoucer - que terá alguns recursos que usaremos;
4. ResoucerExceptionHandler - que estará dentro da pasta Resoucer.
5. StandardError - que também estará dentro da pasta Resoucer.

## Configuração de cada arquivo:

### DatabaseException
```
public class DatabaseException extends RuntimeException{
    private static final long serialVersionUID = 1L;

    public DatabaseException(String msg){
        super(msg);
    }
}
```

### NotFoundException
```
public class NotFoundException extends RuntimeException{
    private static final long serialVersionUID = 1L;

    public NotFoundException(Object id){
        super("Not found resoucer. Id" + id);
    }
}
```

### StandardError
```
public class StandardError implements Serializable {
    private static final long serialVersionUID = 1L;

    private Instant timestamp;
    private Integer status;
    private String error;
    private String message;
    private String path;

    public StandardError() {
    }

    public StandardError(Instant timestamp, Integer status, String error, String message, String path) {
        this.timestamp = timestamp;
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    public Instant getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(Instant timestamp) {
        this.timestamp = timestamp;
    }

    public Integer getStatus() {
        return status;
    }

    public void setStatus(Integer status) {
        this.status = status;
    }

    public String getError() {
        return error;
    }

    public void setError(String error) {
        this.error = error;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getPath() {
        return path;
    }

    public void setPath(String path) {
        this.path = path;
    }
}
```

### ResoucerExceptionHandler
```
// Isso indica que essa classe irá tratar erros especiais.
@ControllerAdvice
public class ResourceExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<StandardError> notFound(NotFoundException e, HttpServletRequest request){
        String error = "Resoucer not found";
        HttpStatus status = HttpStatus.NOT_FOUND;
        StandardError err = new StandardError(Instant.now(), status.value(), error, e.getMessage(), request.getRequestURI());
        return ResponseEntity.status(status).body(err);
    }

    @ExceptionHandler(DatabaseException.class)
    public ResponseEntity<StandardError> database(DatabaseException e, HttpServletRequest request){
        String error = "Database error";
        HttpStatus status = HttpStatus.BAD_REQUEST;
        StandardError err = new StandardError(Instant.now(), status.value(), error, e.getMessage(), request.getRequestURI());
        return ResponseEntity.status(status).body(err);
    }
}
```

Essa classe irá fazer as comunicações entre as mensagens de erro e a sua manipulação com o StandardError.

## Correções nos Services

Após isso, iremos fazer várias correções das chamadas ao repositório. Dentre elas estão:

### delete
```
public void delete(Long id){
        try{
            repository.deleteById(id);
        } catch (EmptyResultDataAccessException e){
            throw new NotFoundException(id);
        } catch (DataIntegrityViolationException e){
            throw new DatabaseException(e.getMessage());
        }
    }
```

### update
```
public Class update(Long id, Products obj){
        try{
            Class entity = repository.getReferenceById(id);
            updateData(entity, obj);
            return repository.save(entity);
        } catch (EntityNotFoundException e){
            throw new NotFoundException(id);
        }
    }
```

### findById (correções)
```
public Class findById(Long id){
    Optional<Class> obj = repository.findById(id);
    return obj.orElseThrow(()-> new NotFoundException(id));
    }
```

Neste último, observe que o obj pode retornar, além do get, outras funções.

---
