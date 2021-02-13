# Intro to Django

## Setup

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

## Create a model

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

go to http://localhost:8000/admin/

## Create api endpoints

install djangorestframework:

```
python -m pip install djangorestframework
```

edit django_rest_api/settings.py

```python
INSTALLED_APPS = [
    'rest_framework',  # add this
    'contacts_api',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

create contacts_api/serializers.py

```python
from rest_framework import serializers 
from .models import Contact 

class ContactSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Contact
        fields = ('id', 'name', 'age',)
```

in django_rest_api/urls.py edit

```python
from django.contrib import admin
from django.urls import path
from django.conf.urls import include # add this

urlpatterns = [
    path('', include('contacts_api.urls')), # add this
    path('admin/', admin.site.urls),
]
```

create contacts_api/urls.py and add

```python
from django.urls import path
from . import views
from rest_framework.routers import DefaultRouter 

urlpatterns = [
    path('api/contacts', views.ContactList.as_view(), name='contact_list'),
    path('api/contacts/<int:pk>', views.ContactDetail.as_view(), name='contact_detail'),
]
```

set contacts_api/views.py to

```python
from rest_framework import generics
from .serializers import ContactSerializer
from .models import Contact

class ContactList(generics.ListCreateAPIView):
    queryset = Contact.objects.all()
    serializer_class = ContactSerializer

class ContactDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Contact.objects.all()
    serializer_class = ContactSerializer
```

## Add CORS
## Switch to Postgres
## Deploy to Heroku
