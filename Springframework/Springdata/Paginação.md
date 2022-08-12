## Paginação

Para que a paginação ocorra de forma correta, devemos fazer algumas modificações no nosso código. Uma delas será o retorno no nosso FindAll. Ao invés de retornamos uma list, iremos retorna uma Page. Além disso, nós iremos por o seguinte código na nossa requisição. Mas, antes de colocar, vejamos os seus parâmetros:

```
@RequestParam(value = "page", defaultValue = "0") Integer page,
// por padrão, o valor default é 0. É por onde iremos começar, se é a página 0 ou outra.
@RequestParam(value = "linesPerPage", defaultValue = "12") Integer linesPerPage,
// Aqui é o valor de linhas por página.
@RequestParam(value = "direction", defaultValue = "DESC") String direction,
// Direção: Asc ou Desc.
@RequestParam(value = "orderBy", defaultValue = "name") String orderBy
// Nome do atributo que nós vamos ordenar os registros. No exemplo dos usuários, retornaremos os nomes.
```

Eles ficarão dentro dos parênteses do findAll. Além disso, iremos implementa mais coisas no corpo do nosso método.
```
public ResponseEntity<Page<User>> findAll(
    @RequestParam(value = "page", defaultValue = "0") Integer page,
    @RequestParam(value = "linesPerPage", defaultValue = "12") Integer linesPerPage,
    @RequestParam(value = "direction", defaultValue = "DESC") String direction,
    @RequestParam(value = "orderBy", defaultValue = "name") String orderBy
){

        PageRequest pageRequest = PageRequest.of(page, linesPerPage, Direction.valueOf(direction), orderBy);
        // Os valores precisam ser, necesessariamente nessa ordem.

        Page<User> list = service.findAllPaged(pageRequest);

        return ResponseEntity.ok().body(list);
    }
```

**Service**

```
@Transactional
public Page<UserDTO> findAllPaged(PageRequest pageRequest){
    Page<User> list = repository.findAll(pageRequest);
	return list.map(x -> new UserDTO(x));
}
```

Ao fazer a requisição no Postman, nós podemos fazer a requisição de outras páginas da seguinte forma:
```
http://localhost/users?page=1
```

Esse "?" serme para colocar parâmetros opcionais. Além disso, podemos trocar os outros parâmetros.
```
http://localhost/users?page=1&linesPerPage=5&direction=ASC&orderBy=name
```

Há uma outra forma muito mais prática para fazer a paginação para a nossa API. Para isso, nós usaremos o a classe Pageable. Ela pertence ao **springframework.data.domain**.

```
public ResponseEntity<Page<User>> findAll(Pageable pageable){
        PageRequest pageRequest = PageRequest.of(pageable);
        Page<User> list = service.findAllPaged(pageRequest);
        return ResponseEntity.ok().body(list);
    }
```

**Service**

```
@Transactional(readOnly = true)
public Page<UserDTO> findAllPaged(Pageable pageable){
    Page<User> list = repository.findAll(pageable);
	return list.map(x -> new UserDTO(x));
}
```

Para usar a requisição, os parâmetros serão um pouco diferentes:

```
http://localhost/users?page=1&size=5&sort=name
```

No final do name, em sort, podemos definir a direção com asc ou desc para queiramos. Basta por uma vírgulo após o name.

```
...sort=name,asc
```

---