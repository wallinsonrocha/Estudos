## Código para colocar no application.properties para fazer a conexão com o banco de dados mysql
```
spring.jpa.hibernate.ddl-auto=update

spring.datasource.url=jdbc:mysql://${MYSQL_HOST:localhost}:3306/db_example
spring.datasource.username=springuser
spring.datasource.password=ThePassword
spring.jpa.properties.hibernate.jdbc.lab.non_contextual_creation=true
```