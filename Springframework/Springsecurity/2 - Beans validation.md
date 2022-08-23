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