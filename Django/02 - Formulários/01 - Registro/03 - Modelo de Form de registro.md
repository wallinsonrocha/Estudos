# Django

## Modelo para forms

Observe como o form pode ser separado.

```
  <div class="main-content center container">
    <h2>Register</h2>
  </div>

  <div class="main-content container">
    <form action="" method="POST">
      {% csrf_token %} # Para evitar espaços em branco e CSRFs

      <div class="form-content form-content-grid">
        {% for field in form %}
          <div class="form-group">
            <label for="{{ field.id_for_label }}">{{ field.label }}</label>
            {{field}}

            {% if field.help_text %}
              <p class="help-text">{{ field.help_text }}</p>
            {% endif %}

            {{ field.errors }}
          </div>
        {% endfor %}
      </div>

      <div class="form-group">
        <button type="submit">Send</button>
      </div>
    </form>
  </div>
```