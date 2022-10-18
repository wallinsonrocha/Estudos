# SQL

## Tipos de dados

### NUMBERS
Serve para armazenar números. Segue exemplo dos mais utilizados

Tipo | Descrição 
-------- | ---------- 
bit | Números inteiros que pode ser 0, 1 ou null
smallint | Permite números inteiros entre -32,768 e 32,767
int | Permite números inteiros entre -2,147,483,648 e 2,147,483,647
bigint | Permite números inteiros entre -9,223,372,036,854,775,808 e 9,223,372,036,854,775,807
float(n) | n é o parâmetro para definir se o campo deve conter 4 ou 8 bytes. float(24) contém um campo de 4 bytes, enquanto float(53) ocupa 8.
real | Precisão de número flutuante de -3,40E + 38 a 3,40E + 38
decimal(e,d) | Define quantas casas para a esquerda e quantas para a direita em relação à virgula.

### STRINGS
Armazena caracteres. Segue alguns exemplos:

Tipo | Descrição 
-------- | ---------- 
char(n) | Tamanho fixo, completado com espaços em bracos.
varchar(n) | Tamanho variável com limite.
varchar(max) | Permite mais do que 8 mil caracteres.
text | Permite 2GB de dados (texto).
enum('a','b','c','um','dois','três'...) | Permite armazenar um valor selecionado de uma lista de valores aceitáveis.

### DATA

Armazena datas. Segue alguns exemplos:

Tipo | Descrição 
-------- | ---------- 
datetime | De 1 de janeiro de 1753 a 31 de dezembro de 9999 com uma precisão de 3,33 milisegundos
datetime2 | De 1º de janeiro de 0001 a 31 de dezembro de 9999 com precisão de 100 nanossegundos
date | Armazena apenas uma data. De 1 de janeiro de 0001 a 31 de dezembro de 9999
time | Armazena um tempo apenas para uma precisão de 100 nanosegundos
datetimeoffset | O mesmo que datetime2 com a adição de um deslocamento de fuso horário
timestamp | Armazena um número único que é atualizado sempre que uma linha é criada ou modificada. O valor do timestamp é baseado em um relógio interno e não corresponde ao tempo real. Cada tabela pode ter apenas uma variável timestamp

### Outros
Há outros tipos, mas os mais utilizados são esses.

---

## Criando banco de dados
Para criar, basta dar o seguinte comando:
```
CREATE DATABASE <nome>;
```

## Para usar o banco de dados
Para usar o banco desejado:
```
USE <nome>;
```

## Criar tabela
Com o banco de dados selecionado< para criar tabela:
```
CREATE TABLE pessoa (
    id INTEGER PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(20) NOT NULL,
    estado CHARACTER(2) NOT NULL
);
```

## Adicionando dados na tabela
Para inserir os dados, nós damos o seguinte comando:
```
INSERT INTO pessoa (nome, nascimento) VALUES ('Wallinson', '2000-08-08');
```
Observe que a sintaze é os atributos que queremos colocar e depois os valores.

## Procurando algo na tabela.
```
SELECT * FROM pessoa;
```

Nesse caso ele busca toda a tabela. Caso seja algo especico de alguém:
```
SELECT * FROM pessoa WHERE id=1;
```

Poderíamos, por exemplo, pegar o nome e o nascimento da pessoa cadastrada:
```
SELECT nome, nascimento FROM pessoa;
```

## Atualizando algo na tabela
```
UPDATE pessoas SET nome = 'Wallinson Rocha' WHERE id=1
```
Nesse caso eu estaria alteando o nome de alguma pessoa que tenha o id=1. Por isso é importante ter a PRIMARYKEY.

## Deletando algo na tabela
```
DELETE FROM pessoas WHERE id=1;
```
É extremamente importante ressaltar que, uma vez apagada, estará apagada para sempre. Não existe lixeira. Por isso, sempre lembre-se de por WHERE.

## Deletando banco de dados

```
DROP DATABASE <nome>;
```
Com esse comando o banco será apagado.

## Ordenando os dados
```
SELECT * FROM pessoa ORDER BY nome;
```
Nesse caso, ao digitar isso, irá trazer todos os nomes em ordem alfabetica. Caso fosse id, ordem numérica.

Caso seja Decrecente, basta adicionar o DESC no final.
```
SELECT * FROM pessoa ORDER BY nome DESC;
```

## Adicionando uma chave ao elemento
```
ALTER TABLE 'pessoa' ADD 'genero' VARCHAR(1) NOT NULL AFTER 'nascimento';
```
Nesse caso eu estou adicionando a chave genero depois nascimento.

## Ordenando a tabela por um valor
Exemplo pego do w3school.
```
SELECT COUNT(CustomerID), Country FROM Customers GROUP BY Country;
```
Selecionando Country em Costumers agrupando-os por Country, nós contamos quantos IDs existem.

---

## Intercessão entre tabelas 
Para isso, necesseitamos de duas tabelas que tenham algum tipo de relacionamento cruzado com os dados. Então, para isso, suponhamos que existem 2 tabelas A e B. Nelas duas foram armazenados chaves "Nome" como parâmetro. E depois adicionamos alguns valores a essa chave.

### Join
```
SELECT * FROM clientes AS c JOIN pedidos AS p ON c.idcliente = p.idpedidos
```
Nesse caso ele irá pegar tudo da tabela cliente e irá fazer a intercessão coma tabela pedidos onde o ID do cliente seja igual ao ID do pedido. Além disso nós podemos fazer a utilização de vários comandos JOIN. Ou seja, o resultado que nós obetivemos no código anterior pode ser utilizado para implementar com outra intercessão. Para isso, basta continuarmos o comando Join.
```
SELECT * FROM clientes AS c JOIN pedidos AS p ON c.idcliente = p.idpedidos
JOIN produtos AS pd ON p.idcliente = pd.idproduto
```

---

## PRIMARY KEY
A primary Key é muito importante ao criar uma tabela. Com a primary key nós podemos identificar algum elemento da tabela com um número que apenas ele tem. Geralmente é utilizado em IDs.
```
CREATE TABLE pessoas (
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    ...
);
```
Ao colocar o PRIMARY KEY e o AUTOINCREMENTE, nós estaremos declarando que o id é uma chave primária e que pode ser autoinplementada com o AUTOINCREMENT.
Adicionando o NOT NULL estaremos dizendo que ele não poderá ser nulo. Ou seja, deve ter, obrigatoriamente, algo nela.

## NOT NULL
O not null indica que essa chave não poderá ser nula. Ou seja, algum valor deverá ser implementada nela.
```
CREATE TABLE pessoas (
    nome VARCHAR(20) NOT NULL
);
```

## AUTO_INCREMENT
Com essa instrução indicamos que aquela chave irá admitir algum valor caso não seja colocado.
```
CREATE TABLE pessoas (
    id INTEGER PRIMARY KEY AUTO_INCREMENT
);
```
