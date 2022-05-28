## Instalação do Webpack
> Obs.: Essa instrução abaixo serve apenas no caso de você não ter usado o "create-reacte-app". Caso você tenha usado, isso pode ser ignorado, porém, algumas configurações e nomes nos script do arquivo package.json, deverão ser mudados.

Antes de seguir os passo abaixo, devemos iniciar o node em nosso projeto criando o arquivo json.
```
npm init
```

Depois iremos cria um arquivo "webpack.config.js" para configurações iniciais. Dentro dele nós iremos por:
```
const path = require('path');

module.exports = {
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, dist),
        filename: 'bundle.js'
    }
}
```

Após isso, para instalá-lo, devemos por no terminal:
```
npm install --save-dev webpack webpack-cli
```

## Adicionando script no packaje.json:
Após isso, no arquivo package.json, devemos adicionar à parte dos scripts o seguinte:
```
"build": "webpack --mode <modo>",
```
> Podemos executar o modo production ou development. Por padrão ele é development. Caso não deseje um mode, podemos por none. Nesse caso, nós iremos usar production.

### Para produção
```
"build": "webpack --mode prodution",
```

Quando executarmos no terminal o:
```
npm run build
```
Ele irá criar a pasta dist onde estará o nosso arquivo bundle.js
Como nós colocamos ele em modo de produção, o arquivo gerado não será para executar testes, debugar ou coisas do tipo. Ele serve apenas para ser colocado em produção.

### Para desenvolvimento
```
"dev": "webpack --mode development",
```

Quando executarmos no terminal o:
```
npm run dev
```
Ele irá criar a pasta dist onde estará o nosso arquivo bundle.js
Como nós colocamos ele em modo de desenvolvimento, o arquivo gerado será para executar testes, debugar ou coisas do tipo.

### Plugin do HTML
Antes de seguir os próximos passos, devemos adicionar o pacote do html colocando no terminal:
```
npm install --save-dev html-webpack-plugin
```

Para adicionar os plugins, nós colocamos ele no module.exports. Isso será adicionado no arquivo "webpack.config.js". Além disso, deve se criado uma constante do módulo html:
```
...
const HtmlWebPackPlugin = require("html-webpack-plugin");

module.export = {
    ...,
    plugins: [
        new HtmlWebPackPlugin({
            template: "./src/index.html",
            filename: "./index.html"
        })
    ]
}
```

## Instalação do babel:
### Caso não tenha adicionado o babel antes:
No terminal:
```
npm install --save-dev @babel/cli @babel/preset-env @babel/core @babel/preset-react
```

### Criação do .babelrc
Como nós utilizamos o preset-env e o preset-react, devemos criar um arquivo chamado ".babelrc" para ser lido pela nossa webpack. E, dentro do arquivo criado, nós iremos adicionar:
```
{
    "presets": [
        "@babel/preset-env",
        "@babel/preset-react"
    ]
}
```

### Adicionando module na webpack
Após isso, no arquivo webpack.config.js, após "output: {...}, " nós iremos adicionar o module:
```
...,
module: {
    rules: [
        {
            teste: /\.(js|jsx)$/,
            exclude: /node_modules/,
            use: {
                loader: "babel-loader"
            }
        }
    ]
}
```

## Criando um Dev Server
Serve para ser a nossa aplicação funcionando sem criar um servidor.

No terminal:
```
npm i -D webpack-dev-server
```

E, nos scripts do package.json:
```
...,
"start:dev": "webpack-dev-server"
```
