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
