# Deploy

## Criando os repositórios da aplicação e bare
Entre [aqui](https://github.com/luizomf/curso-django-projeto1/tree/6040aa148b59551232a66228b732b2a2c569538e/deploy#criando-um-reposit%C3%B3rio-no-servidor) para seguir o tutorial.

## Criando .env no servidor
Após isso, caso ocorra algum erro ao migrar os dados, nós devemos criar um arquivo `.env` e, através do nano, configurá-lo.

Copiando o .env-example para o .env
```
cp .env-example .env
```

Após isso basta entrar e modificar aquilo que será necessário.
```
nano .env
```

## Preparando o site para o servidor
Esse processo é o mesmo quando ativamos em nosso servidor local utilizando o manage.py.