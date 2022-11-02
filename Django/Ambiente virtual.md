# Django

## Ambiente virtual
O ambiente virtual serve para configurar um ambiente de trabalho. Através dela evitamos o risco de quebrar alguma aplicação por conta de alguma atualização. Além disso, podemos fazer a instalação de extenções/módulos/bibliotecas para determinado ambiente.

## Criando em Python
Na distro baseada no Ubunto é necessário instalar o venv. Este código abaixo serve para intalar.
```
apt install python3.8-venv -y
```

### Para criar um ambiente virtual.
```
python3 -m venv venv
```

### Para ativar e desativar
```
source venv/bin/activate
```
Para desativa basta digitar **deactivate**.

### Automatizando pelo VS Code
Após criar o ambiente virtual, apertamos **ctrl+shift+p** e digitar **Select Interpreter** e selecionar a versão do Python recomendada.