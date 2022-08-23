## Estrutura básica

## Recomendação para criação de pacotes

Recomenda-se a criação dos seguintes pacotes para que haja melhor manutenção do código e organização:

1. Config - Configuração
2. Entities - As tabelas
3. Repositories - Para o repositório
4. Services - Para adicionar as services
5. Controllers ou Resources - Controles.
6. Exceptions - Para tratar exeções.

Caso seja NoSQL, no lugar de Entities, colocamos "Domain".

### Entity

```
@Entity
@Table(name = "tb_value")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    private String password;
}
```

### Repository

```
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
}
```

### Service

```
@Service
public class UserService {

    @Autowired
    private UserRepository repository;

    public List<User> findAll(){
        return repository.findAll();
    }
    
    public User findById(@PathVariable Long id){
        Optional<User> obj = repository.findById(id);
        return obj.get();
    }
}
```

### Controller

```
@RestController
@RequestMapping(value = "/users")
public class UserController {
    @Autowired
    private UserService service;

    @GetMapping
    public ResponseEntity<List<User>> findAll(){
        User u1 = new User(null, "Carlos", "carlos@email.com", "123456");
        List<User> list = service.findAll();
        return ResponseEntity.ok().body(list);
    }

    @GetMapping(value = "/{id}")
    public ResponseEntity<User> findById(@PathVariable Long id){
        User obj = service.findById(id);
        return ResponseEntity.ok().body(obj);
    }
}
```

---
