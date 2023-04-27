# Aula 3
Essa é a parte báscica do git.
Antes de iniciarmos, vamos criar uma pasta para colocar o nosso arquivo que iremos colocá-lo no nosso repositório do github. Em seguida, vamos criar um arquivo chamado README.md.

## Configuando o git
Antes de usarmos o git, nós podemos configurá-lo.

### Usuário global
```
git config --global user.name "nome"
```

### Adicionando o seu email
```
git config --global user.email "email@email.com"
```

### Conectar com o Editor de Código
```
git config --global core.editor vscode
```

### Para ver tudo que está cadastrado
```
git config --list
```

---

## Comandos básicos
### git init
Vamos abrir o terminal e navegar até a pasta do projeto. Se você usa linux mint, você pode, já na pasta do projeto, clicar com o botão direito e selecionar em "Abrir no terminal".

Esse comando serve para iniciar o git em nossa pasta. Com ela será reconhecida o git. Quando executarmos ela, uma pasta oculta .git será criada.

```
git init
```

### git remote
Após a execução no git init, iremos criar um repositório remoto em nosso pc. Ela estará ligada ao repositório que nós queremos ligar.

Lembra daquele link ssh (não é a chave, ok?) que criamos na aula 2 ao criar o repositório no github? Nós iremos copiar ela para executar esse comando.

```
git remote add origin <link ssh>
```

Caso quisermos deletar ela, podemos usar
```
git remote remove origin <branch>
```
> Nós iremos aprender o que é branch.

Para verificar se foi criada ou quantos repositórios remotos existem, basta dar um "git remote". Geralmente é feita apenas uma, que é a origin no caso, mas podem ser feitas outras com outros nomes além do nome criado.

### git status
Para verificar a condição e o status do nosso repositório remoto, podemos usar:
```
git status
```
Através dela nós iremos ver se algum aquivo foi alterado ou se precisamos adicionar algo.

### git branch
Depois disso precisamos criar uma branch, que serão os nós. Geralmente eles são usados para diferenciar uma versão da outra ou trabalhar em uma versão diferente da branch principal. Elas podem ter qualquer nome. No nosso caso iremos usar master.

```
git branch master
```

Para mudar de branch, caso haja outras:
```
git checkout <branch>
```
Assim iremos carregar os arquivos que existem nessa branch.

Para se localizar (saber em que branch está):
```
git branch
```

Para deletar ela no repositório remoto:
```
git branch -D <branch>
```

Caso seja necessário deletar ela no repositório do github:
```
git push origin :<branch>
```

### git add
Após isso, precisamos adicionar os arquivos na nossa branch do repositório remoto. Caso a branch não seja criada antes, será criada uma automaticamente ao subir os arquivos para o github com o nome de "master".

Caso deseje adicionar todos os arquivos no repositório remoto.
```
git add -A ou * ou .
```
Geralmente é utilizado.

Caso queira adicionar um arquivo específico.
```
git add <nome dos arquivos separados por espaço com a extensão(.js, .ts e etc) deles>
```

### git commit
Os commits no git servem para armazenar anotações do que foi feito nos arquivos e salvá-los no repositório remoto para enviar ao github. Para testar, escreva alguma coisa no arquivo README que nós criamos. Em seguida, com o git status, veja o que aparece. Após isso, você pode executar o comando abaixo no terminal.
```
git commit -m "Anotação"
```

Caso seja modificado um aquivo ou deletado, podemos usar o -am. Geralmente é utilizado para não usar o git add. Eu costumo o -am para todos os meus commits.
```
git commit -am "Anotação"
```

### git push
Quando criamos um repositório remoto e não subimos ele ao github nenhuma vez, usamos:
```
git push -u origin <branch>
```
> Geralmente, quando não criamos uma branch antes e especificamos ela ao dar o push, ela será criada no repositório remoto e no github

Quando queremos atualizar ele depois de subí-lo no github:
```
git push origin master
```

Após isso, você já poderá ver o seu arquivo lá no repositório do Github.

### git pull
Usado para pegar as modificações que foram feitas no repositório do GitHub. Suponhamos que fora criado um repositório onde nós armazenamos os nossos arquivos usando o push. Mas, por algum motivo, depois de suas alterações salvas, ela recebeu outra alteração que não foi fe3ita pelo seu repositório remoto. Para pegá-las, você deve dar um pull.
```
git pull origin master
```

## Correção caso o .gitignore não esteja funcionando
```
git rm -r --cached .
```
