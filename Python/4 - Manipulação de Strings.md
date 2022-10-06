# Python

## Manipulação de Strings

### Maiúscula, minúscula e título

```
curso = "pYthon"

print(curso.upper())
// PYTHON

print(curso.lower())
// python

print(curso.title())
// Python
```

### Eliminar espaços em branco
```
curso = " Python "

print(curso.strip())
// "Python"

print(curso.lstrip())
// "Python "

print(curso.rstrip())
// " Python"
```

### Junções e centralização
```
curso = "Python"

print(curso.center(10, "#"))
// "##Python##"
// Nesse caso ele irá preencher os espaços até que tenha 20 caracteres.

print(".".join(curso))
// "P.y.t.h.o.n"
```