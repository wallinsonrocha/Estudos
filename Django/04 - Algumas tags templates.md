# Django

## Herança 
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

Podemos cortar parte do código e por nesse arquivo criando uma herança para extender para home.html.
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

