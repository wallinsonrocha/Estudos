# Django

## Mudando o Time Zone, tradução e Intercionalização

No arquivo `settings.py` do nosso projeto podemos ir até # Internationalization e modificar algumas coisas de acordo com nosso preferência.

```
# Internationalization
# https://docs.djangoproject.com/en/3.2/topics/i18n/

LANGUAGE_CODE = 'pt-br'

TIME_ZONE = 'America/Sao_Paulo'

USE_I18N = True

USE_L10N = True

USE_TZ = True
```