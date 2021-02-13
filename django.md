```
python3 -m venv tutorial-env
source tutorial-env/bin/activate
python -m pip install Django
django-admin startproject django_rest_api
cd django_rest_api
python manage.py startapp contacts_api
python manage.py migrate
```

edit django_rest_api/settings.py

```python
INSTALLED_APPS = [
    'contacts_api', # add this
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
````

add to contacts_api/models.py

```python
class Contact(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
```

terminal

```
python manage.py makemigrations contacts_api
python manage.py sqlmigrate contacts_api 0001
python manage.py migrate
```

enter into shell

```
python manage.py shell
```

in shell

```python
from contacts_api.models import Contact
Contact.objects.all()
c = Contact(name="Matt", age=40)
c.save()
c.id #should return 1
Contact.objects.all()
quit()
```

in terminal and follow prompts

```
python manage.py createsuperuser
```

add to contacts_api/admin.py

```python
from .models import Contact
admin.site.register(Contact)
```

in terminal

```
python manage.py runserver
```

go to http://127.0.0.1:8000/admin/
