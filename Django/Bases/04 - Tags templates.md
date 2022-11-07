# Django

## Herança 

Antes de partir para os exemplos, não podemos esquecer de tornar a pasta **base_template** reconhecida. Para isso, em settings, na parte de **TEMPLATES** e, mais especificamente em **'DIR'**, iremos adicionar:
```
'DIRS': [
            BASE_DIR / 'base_templates', # Parte adicionada
        ],
```
> Lembre-se que o base_template deve ser criado fora da aplicação.


Vamos supor que na nossa página o home.html estava assim:
```
{% include 'recipes/partials/head.html' %}

<body>
    {% include 'recipes/partials/header.html' %}
    {% include 'recipes/partials/search.html' %}

    <main class="main-content-container">
        <div class="main-content main-content-list container">
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
            {% include 'recipes/partials/recipe.html' %}
        </div>
    </main>

    {% include 'recipes/partials/footer.html' %}
</body>

</html>
```

No nosso projeto, em um determinado ponto, podemos ter elementos em comum em várias páginas HTML. Para reutilizar esses elementos em comum podemos criar uma pasta **base_templates** e criar um base.

Podemos cortar parte do código e por nesse arquivo criando uma herança para extender em home.html.
```
{% include 'recipes/partials/head.html' %}

<body>
    {% include 'recipes/partials/header.html' %}
    {% include 'recipes/partials/search.html' %}

    <main class="main-content-container">
        {% block content %}{% endblock content %} # importante
    </main>

    {% include 'recipes/partials/footer.html' %}
</body>

</html>
```

Observe que há um **{% block name %}{ % endbloc name %}**. Ele será responsável por coletar as extenções que estarão em home.html (modificação abaixo) e colocar sempre que home for ecessado.
```
{% extends 'global/base.html' %} # importante

{% block content %} # importante
<div class="main-content main-content-list container">
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
    {% include 'recipes/partials/recipe.html' %}
</div>
{% endblock content %} # importante
```

Isso irá fazer com que os conteúdos de **home** que estão dentro do bloco sejam colocados em **base** sempre que home for acessado.

Seria muito importante dividir o nosso código no **base.html** dessa forma:
```
<head>
    {% include 'recipes/partials/head.html' %}
</head>
<body>
    {% include 'recipes/partials/header.html' %}  
    <main>
        {% block section %}{% endblock section %}
    </main>
    <footer>
        <img src="./assets/1b-arquivos-necessarios-criando-topo/img/fundo-rodape.png">
        <ul>
            <li><a href="#">HOME</a></li>
            <li><a href="#">EXPOSIÇÕES</a></li>
            <li><a href="#">PESQUISA</a></li>
            <li><a href="#">ACERVO</a></li>
            <li><a href="#">VÍDEOS</a></li>
            <li><a href="#">FOTOS</a></li>
            <li><a href="#">CONTATO</a></li>
        </ul> 
        <p>2019 <a href="#">Museu Nacional</a> - Todos os direitos reservados.</p>
    </footer>

    <script type="module" src="./assets/javascript/main.js"></script>
</body>

</html>
```

Assim estará bem dividido com head e body.

## If, else, for e url

Assim como o **include** tem a sua tag, o if, else e o for também tem.
```
# IF e ELSE
{% if condição %}
    ...
{% else %}
    ...
{% endif  %}

#FOR
{% for variavel in lista/obejeto %}
    ...
{% endfor %}
```

A **Url** é usada no html da nossa aplicação.
```
<footer>
        <img src="{% static 'recipes/partials/recipe.jpg' %}">
        <ul>
            <li><a href="{% url 'recipes:category' recipe.category.id %}">HOME</a></li>
        </ul> 
        <p>2019 <a href="#">Museu Nacional</a> - Todos os direitos reservados.</p>
    </footer>
```

É importante ressaltar que em **'recipes:category'**, **category** refere-se ao nome que atribuimos na criação da **url de catergory**.

## for empaty
Semelhante ao if mas para o for. Isso ajudará a poupar linhas de código.

```
{% for recipe in recipes %}
    {% include 'recipes/partials/recipe.html' %}
{% empty %}
    <div class="center m-y">
        <h1>No recipes found here 🥲</h1>
    </div>
{% endfor %}
```

## safe e linebreaksbr
Permissão para renderizar ou não o HTML de um determinado conteúdo.

```
{% if is_detail_page is True %}
    <div class="preparation-steps">
        {% if recipe.preparation_steps_is_html is True %}
            {{ recipe.preparation_steps|safe }}
        {% else %}
            {{ recipe.preparation_steps|linebreaksbr }}
        {% endif %}
    </div>
{% endif %}
```

No exemplo acima diz que, se tal texto não for html, ele não será renderizado como html, mas como texto. Caso contrário, se for, ele irá renderizar como hmlt.

**linebreaksbr** - Renderiza como texto.
**safe** - Renderiza como html.