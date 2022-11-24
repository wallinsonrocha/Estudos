# Django

```
class Meta:
    model = User
    fields = [
        'first_name',
        'last_name',
        'username',
        'email',
        'password',
    ]
```

## Labels
```
labels = {
    'username': 'Username',
    'first_name': 'First name',
    'last_name': 'Last name',
    'email': 'E-mail',
    'password': 'Password',
    }
```

## Help texts
```
help_texts = {
    'email': 'The e-mail must be valid.',
    }
```

## Error messages
```
error_messages = {
    'username': {
        'required': 'This field must not be empty',
    }
}
```

