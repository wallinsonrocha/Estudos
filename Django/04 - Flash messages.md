# Django

## Utilização
Geralmente colocada em alguma função no arquivo `view.py`. A home, por exemplo.
```
from django.contrib import messages
...

def home(request):
    recipes = Recipe.objects.filter(
        is_published=True,
    ).order_by('-id')



    #Parte implementada
    messages.success(request, 'Epa, você foi pesquisar algo que eu vi.')




    page_obj, pagination_range = make_pagination(request, recipes, PER_PAGE)

    return render(request, 'recipes/pages/home.html', context={
        'recipes': page_obj,
        'pagination_range': pagination_range
    })
```

### Implementação na pagina que desejamos expor a mensagem
**Partial para messages**
```
{% if messages %}
<div class="main-content center container messages-container">
    {% for message in messages %}
        <div class="message {{ message.tags }}">
            {{ message }}
        </div>
    {% endfor %}
</div>
{% endif %}
```

**Para colocar na página que queremos como a home.html**
```
{% include 'recipes/partials/messages.html' %}
```

O django reconhece automaticamente caso haja alguma mensagem. Por isso que no if há a variável **messages**.

## Modificação no `settings.py` para o CSS
```
from django.contrib.messages import constants
...

MESSAGE_TAGS = {
    constants.DEBUG: 'message-debug',
    constants.ERROR: 'message-error',
    constants.INFO: 'message-info',
    constants.SUCCESS: 'message-success',
    constants.WARNING: 'message-warning',
}
```

Através dessa constante do django iremos configurar as nossas classes CSS para as mensagens.

## CSS
```
:root {
  --color-primary: #269fe6;
  --color-primary-hover: #2086c2;
  --color-primary-dark: #13141f;
  --color-primary-dark-hover: #212336;
  --color-primary-light: #d4ecfa;
  --color-primary-light-hover: #bdd8e7;

  --color-white: #fff;
  --color-black: #000;

  --color-dark-text: #444;
  --color-info-light: #cce5ff;
  --color-debug-light: #cce5ff;
  --color-success-light: #d4edda;
  --color-alert-light: #fff3cd;
  --color-warning-light: #fff3cd;
  --color-error-light: #f8d7da;

  --color-info-dark: #4d86c4;
  --color-debug-dark: #4d86c4;
  --color-success-dark: #4a9c5d;
  --color-alert-dark: #927f40;
  --color-warning-dark: #927f40;
  --color-error-dark: #da525d;

  --color-gray-0: #f9f9f9;
  --color-gray-1: #e0e0e0;
  --color-gray-2: #c7c7c7;
  --color-gray-3: #aeaeae;
  --color-gray-4: #959595;
  --color-gray-5: #7d7d7d;
  --color-gray-6: #646464;
  --color-gray-7: #4b4b4b;
  --color-gray-8: #323232;
  --color-gray-9: #191919;

  --font-primary: sans-serif;
  --font-headings: 'Roboto Slab', serif;

  --spacing-gutter-medium: 3rem;
  --spacing-gutter-large: 4rem;
}


/* Messages */
.messages-container {
  display: flex;
  flex-flow: column nowrap;
  gap: calc(var(--spacing-gutter-medium) / 2);
}

.message {
  padding: 1rem;
  border-radius: 4px;
  border: 1px solid var(--color-dark-text);
  background: var(--color-gray-2);
  color: var(--color-dark-text);
}

.message-error {
  border: 1px solid var(--color-error-dark);
  background: var(--color-error-light);
  color: var(--color-error-dark);
}

.message-success {
  border: 1px solid var(--color-success-dark);
  background: var(--color-success-light);
  color: var(--color-success-dark);
}

.message-warning {
  border: 1px solid var(--color-warning-dark);
  background: var(--color-warning-light);
  color: var(--color-warning-dark);
}

.message-alert {
  border: 1px solid var(--color-alert-dark);
  background: var(--color-alert-light);
  color: var(--color-alert-dark);
}

.message-info {
  border: 1px solid var(--color-info-dark);
  background: var(--color-info-light);
  color: var(--color-info-dark);
}

.message-debug {
  border: 1px solid var(--color-debug-dark);
  background: var(--color-debug-light);
  color: var(--color-debug-dark);
}

/* Para o mobile */
@media (max-width: 600px) {
  .main-content-list {
    grid-template-columns: repeat(auto-fit, minmax(100%, 1fr));
  }
}
```