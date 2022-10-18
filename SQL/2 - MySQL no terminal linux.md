# SQL

Dentre os instaladores que temos, o lampp é uma alternativa exclusiva ao linux. Com esse pacote, assim como os demais, nós temos acesso ao mysql do terminal.

## Acessando o mysql no terminal
Abra o seu terminal com Ctrl + Alt + t. Em seguida, rode o comando:
```
sudo mysql
```

Esse comando será usado quando você tiver usando o MySQL pela primeira vez.

Já nas demais vexes, entraremos colocando o nosso usuário e depois a senha:
```
mysql -u <user> -p
```

## Criando usuário e senha.
Logo após, você irá precisar criar um usuário e senha para acessar o seu MySQL. Para isso você irá colocar e editar no terminal o seguinte cógigo:
```
CREATE USER '<novo_usuário>'@'localhost' IDENTIFIED BY '<senha>'
```

---

## Garantindo privilégios ao usuário criado
Essa etapa é importante para fazer a criação de bancos e manipulação.
```
GRANT ALL PRIVILEGES ON <nome_do_banco_de_dados>.<nome_da_tabela> TO '<usuario>'@'localhost';
```

Geralmente é utilizado * que significa todos no lugar de \<nome_do_banco_de_dados\> e \<nome_da_tabela\>

Para que as mudanças tenham efeito, execute imediatamente um flush dos privilégios ao executar o seguinte comando:
```
FLUSH PRIVILEGES;
```

### Garantindo alguns privilégios
Caso o seu desejo seja conceder alguns privilégios a algum usuário, nós devemos listar eles. Nós temos: CREATE, SELECT, INSERT, UPDATE, DELETE, DROP.
```
GRANT SELECT, INSERTE ON * . * TO '<usuário>'@'localhost';
```

### Mostrar os privilégios
Para observar os privilégios concedidos a algum usuário, nós podemos usar:
```
SHOW GRANTS FOR '<usuário>'@'localhost';
```

### Removendo privilégios
Pode ser que tenhamos que passar por essa situação. Logo, a remoção de privilégios é:
```
REVOKE [tipoS de priviégio] ON <banco de dados>.<tabela> FROM '<usuários>';
```

Exemplo:
```
REVOKE ALL PRIVILEGES ON *.* FROM '<usuários>';
```

### Removendo usuário
Finalmente você pode excluir completamente um usuário existente:
```
DROP USER '<usuário>'@'localhost';
```

---

## Alguns comandos
### Listar os banco de dados
Abra o terminal acesso o MySQL e digite:
```
show databases;
```

### Selecionando banco de dados
Para selecionar o bando de dados:
```
USE <banco de dados>;
```

### Listar as tabelas do banco de dados
Abra o terminal acesso o MySQL, em seguida acesse o banco de dados desejado e digite:
```
show tables;
```

### Visualizar estrutura da tabela

Abra o terminal acesso o MySQL, em seguida acesse o banco de dados desejado e digite:
```
describe <nome da tabela>;
```
