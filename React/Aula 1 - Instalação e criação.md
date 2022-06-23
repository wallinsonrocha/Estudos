# Aula 1

## Instalação do Node e do Yarn
Para instalar podermos utilizar a biblioteca do React JS, primeiro precisamos instalar a versão mais recente  no NodeJS na nossa máquina. Esse processo é simples. Podemos acessar o [site oficial](https://nodejs.org/en/download/). Nele vamos encontrar o download para Windows e Mac.

Para fazer a instalação no Linux, podemos seguir esse [tutorial](https://www.treinaweb.com.br/blog/instalacao-do-node-js-windows-mac-e-linux).

Ao fazer a instalação no NodeJS, o NPM também será instalado. Caso queira utilizar o Yarn - que é recomendável -, devemos instalar as suas dependências. Para isso, devemos abrir o terminal e digitar:

```
npm install -g yarn
```

Por opção, escolhi usar o npm.

## Instalação do React JS
Para instalá-lo, devemos abrir o terminal e, com o npm ou yarn (de acordo com a sua preferência), digitamos:

```
npm install -g create-react-app
```

Após a sua instalação, nós temos acesso à criação da aplicação do ReactJS

## Criando App
Para criamos a aplicação, recomenda-se criar uma pasta onde os seus projetos ficarão armazenados. Após criar tal pasta, abriremos o terminal e iremos até a pasta criada. Quem usa linux mint basta escolher a opção de abrir o terminal na pasta que você está.
No terminal criaremos assim:
```
npx create-react-app meu-app
```

É "x", mesmo "npx".

O nome da aplicação - a pasta que será criada - não deverá ter letras maiúsculas nem espaço. Nós podemos separar por "-" ou por "_".

Para rodar a aplicação, iremos, no terminal, até a pasta criada e digitar:
```
npm start
```