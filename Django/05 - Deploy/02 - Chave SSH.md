# Deploy

## Observações
Essas consfigurações só podemram ser completamente executadas caso já tenhamos criado um servidor em alguma plataforma. Poderá ser Google Cloud, AWS, Heruko, LocalWeb, Hostinger e etc. Isso é necessário para que possamos utilizar o IP externo.

## Coletando chave pública
Utilizando o comando abaixo, podemos coletar a nossa chave SSH criada através do Git.
```
cat .ssh/id_<nomedachave>.pub
```
Após isso, basta adicionar essa chave pública na nossa plataforma de preferência. No Google Cloud está em Computer Engine > Metadados > Chave SSH.

## Conectando com o servidor
Para conectar no servidor basta executarmos o código abaixo.
```
ssh <usuario>@<ip externo copiado>
```

Caso seja uma chave ssh personalizada, podemos indicar o seu caminho.
```
ssh <usuario>@<ip externo copiado> -i '~/.ssh/<nome da chave>'
```

> Preferencialmente, se é necessário fazer isso, é melhor que configure.

Podemos seguir esse [tutorial](https://github.com/luizomf/curso-django-projeto1/tree/f72381b86b5b28740d4310a58b8e6697703dfceb/deploy#chaves-ssh).