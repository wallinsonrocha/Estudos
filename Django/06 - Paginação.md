# Django

## Paginator
- [Documentação](https://docs.djangoproject.com/en/3.2/topics/pagination/)

Caso seja necessário criar uma **paginação personalizada**, cria-se uma pasta chamada `utils` para colocar a função e o teste.

`pagination.py`
```
from django.core.paginator import Paginator
import math


def make_pagination_range(
    page_range,
    qty_pages,
    current_page,
):
    middle_range = math.ceil(qty_pages / 2)
    start_range = current_page - middle_range
    stop_range = current_page + middle_range
    total_pages = len(page_range)

    start_range_offset = abs(start_range) if start_range < 0 else 0

    if start_range < 0:
        start_range = 0
        stop_range += start_range_offset

    if stop_range >= total_pages:
        start_range = start_range - abs(total_pages - stop_range)

    pagination = page_range[start_range:stop_range]
    return {
        'pagination': pagination,
        'page_range': page_range,
        'qty_pages': qty_pages,
        'current_page': current_page,
        'total_pages': total_pages,
        'start_range': start_range,
        'stop_range': stop_range,
        'first_page_out_of_range': current_page > middle_range,
        'last_page_out_of_range': stop_range < total_pages,
    }


def make_pagination(request, queryset, per_page, qty_pages=4):
    try:
        current_page = int(request.GET.get('page', 1))
    except ValueError:
        current_page = 1

    paginator = Paginator(queryset, per_page)
    page_obj = paginator.get_page(current_page)

    pagination_range = make_pagination_range(
        paginator.page_range,
        qty_pages,
        current_page
    )

    return page_obj, pagination_range
```

`test_pagination.py`
```
from unittest import TestCase

from utils.pagination import make_pagination_range


class PaginationTest(TestCase):
    def test_make_pagination_range_returns_a_pagination_range(self):
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=1,
        )['pagination']
        self.assertEqual([1, 2, 3, 4], pagination)

    def test_first_range_is_static_if_current_page_is_less_than_middle_page(self):  # noqa: E501
        # Current page = 1 - Qty Page = 2 - Middle Page = 2
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=1,
        )['pagination']
        self.assertEqual([1, 2, 3, 4], pagination)

        # Current page = 2 - Qty Page = 2 - Middle Page = 2
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=2,
        )['pagination']
        self.assertEqual([1, 2, 3, 4], pagination)

        # Current page = 3 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=3,
        )['pagination']
        self.assertEqual([2, 3, 4, 5], pagination)

        # Current page = 4 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=4,
        )['pagination']
        self.assertEqual([3, 4, 5, 6], pagination)

    def test_make_sure_middle_ranges_are_correct(self):
        # Current page = 10 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=10,
        )['pagination']
        self.assertEqual([9, 10, 11, 12], pagination)

        # Current page = 14 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=12,
        )['pagination']
        self.assertEqual([11, 12, 13, 14], pagination)

    def test_make_pagination_range_is_static_when_last_page_is_next(self):
        # Current page = 18 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=18,
        )['pagination']
        self.assertEqual([17, 18, 19, 20], pagination)

        # Current page = 19 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=19,
        )['pagination']
        self.assertEqual([17, 18, 19, 20], pagination)

        # Current page = 20 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=20,
        )['pagination']
        self.assertEqual([17, 18, 19, 20], pagination)

        # Current page = 21 - Qty Page = 2 - Middle Page = 2
        # HERE RANGE SHOULD CHANGE
        pagination = make_pagination_range(
            page_range=list(range(1, 21)),
            qty_pages=4,
            current_page=21,
        )['pagination']
        self.assertEqual([17, 18, 19, 20], pagination)
```

## Implementação na view.
Para implementar na view, devemos seguir o seguinte exemplo:
```
...

def home(request):
    recipes = Recipe.objects.filter(
        is_published=True,
    ).order_by('-id')

    current_page = request.GET.get('page', 1)
    paginator = Paginator(recipes, 9)
    page_obj = paginator.get_page(current_page)

    return render(request, 'recipes/pages/home.html', context={
        'recipes': page_obj,
    })
```

ou seguindo a implementação da função de **paginação personalizadas**.

```
from utils.pagination import make_pagination
...

PER_PAGES = 9


def home(request):
    recipes = Recipe.objects.filter(
        is_published=True,
    ).order_by('-id')

    page_obj, pagination_range = make_pagination(request, recipes, PER_PAGES)

    return render(request, 'recipes/pages/home.html', context={
        'recipes': page_obj,
        'pagination_range': pagination_range
    })
```

Após isso, podemos criar uma **partial** chamada `pagination.html` e implementar o código conforme o exemplo.

```
{% if recipes.has_other_pages %}
  <div class="container pagination">
    <div class="pagination-content">
      {% if pagination_range.first_page_out_of_range %}
        <a class="page-link page-item" href="?page=1{{ additional_url_query }}">1</a>
        <span class="page-item">...</span>
      {% endif %}

      {% for page in pagination_range.pagination %}
        {% if pagination_range.current_page == page %}
          <a class="page-link page-item page-current" href="?page={{ page }}{{ additional_url_query }}">{{ page }}</a>
        {% else %}
          <a class="page-link page-item" href="?page={{ page }}{{ additional_url_query }}">{{ page }}</a>
        {% endif %}
      {% endfor %}

      {% if pagination_range.last_page_out_of_range %}
        <span class="page-item">...</span>
        <a class="page-link page-item" href="?page={{ pagination_range.total_pages }}{{ additional_url_query }}">{{ pagination_range.total_pages }}</a>
      {% endif %}    
    </div>
  </div>
{% endif %}
```

Após isso basta incluir esse código em nossa `base.html` na **base_template**.
```
<!DOCTYPE html>
<html lang="pt-BR">

<head>
    {% include 'recipes/partials/head.html' %}
    <title>{% block title %}{% endblock title %} Recipes</title>
</head>

<body>
    {% include 'recipes/partials/header.html' %}
    {% include 'recipes/partials/search.html' %}

    <main class="main-content-container">
        {% block content %}{% endblock content %}
        {% include 'recipes/partials/pagination.html' %} # Parte adicionada
    </main>

    {% include 'recipes/partials/footer.html' %}
</body>

</html>
```