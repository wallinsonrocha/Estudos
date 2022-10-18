# SQL

## Constraints
As constraints servem para fazer validações dos dados.

```
CREATE TABLE table_name (
    column1 datatype NOT NULL,
    column2 datatype UNIQUE,
    column3 datatype PRIMARY KEY,
    column3 datatype FOREIGN KEY,
    ....
);
```

### NOT NULL
Indica que certo dado não poder ser nulo.

### UNIQUE
Para demonstrar que o caractere deve ser único. Não pode haver outro igual.

### PRIMARY KEY
Geralmente colocada ao lado do dado que representará um ID. É uma combinação de NOT NULL e UNIQUE.

### FOREIGN KEY
Impede ações que destruiriam links entre tabelas. (Ainda faltam informações. Continur aqui...)

### PK & SK