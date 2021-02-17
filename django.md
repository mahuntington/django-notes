# Intro to Django

## Setup

Let's create a special development environment.  This will separate what we use for class from the rest of your system.  It also makes installing python packages easier

```
python3 -m venv ~/ga-env
```

now that it's create it, let's start it up:

```
source ~/ga-env/bin/activate
```
**NOTE:** you'll have to run `source ~/ga-env/bin/activate` every time you create a new terminal window.  If you want, you can put this command in `~/.bash_profile` or `~/.zshenv` depending on whether you're using bash or zsh, respectively

Now let's install Django.  This will allow us to create/run django apps:

```
python -m pip install Django
```

Let's create a new django project.  Go to where on your computer you want your app to be stored and run:

```
django-admin startproject django_rest_api
```

This is kind of `npm init`.  Now, go run

```
cd django_rest_api
python manage.py startapp contacts_api
```

This will move into your project dir and create an app called `contacts_api`.  A django project can contain many apps.  Each app is a logical section of your that is self contained.  It's a bit like a controller file in express which contains all routes for one specific model

Now let's get Postgres hooked up to Django.  Start your postgres server, open Postgres, and choose any sub database.  Once in there, create a sub database that our project will use:

```
CREATE DATABASE django_contacts;
```

Now edit django_rest_api/settings.py:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'django_contacts',
        'USER': '',
        'PASSWORD': '',
        'HOST': 'localhost'
    }
}
```

back in terminal run

```
python -m pip install psycopg2
```

This installs a driver that allows Django to talk to Postgres.  It's a bit like Mongoose.

Now we want to run a migration to set up the tables necessary to get django working.  Migrations are python files that run SQL for you, so that you don't have to write it yourself

```
python manage.py migrate
```

Now that the db is set up, let's register our contacts_api with django.  This is a bit like in express when we require a controller file into server.js

edit django_rest_api/settings.py:

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

Now let's create a model.  This is similar to migrations, in that it allows us to write python code that will handle the writing of SQL for us.

add to contacts_api/models.py:

```python
class Contact(models.Model):
    name = models.CharField(max_length=32)
    age = models.IntegerField()
```

Now let's set up a migration that will access our new `Contact` model and generate the necessary table in Postgres.  In the terminal, run:

```
python manage.py makemigrations contacts_api
```

This creates the migration, but doesn't execute it.  If we want, we can see what sql will be run:

```
python manage.py sqlmigrate contacts_api 0001
```

Note, if you create more migrations later on, you'll have to update `0001` to the number of the migration file that was created (check in `contacts_api/migrations/` for the appropriate `.py` file)

Now let's have django run any outstanding migrations that haven't been run yet:

```
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

```
python -m pip install django-cors-headers
```

edit django_rest_api/settings.py

```python
INSTALLED_APPS = [
    'corsheaders', # add this
    'rest_framework',
    'contacts_api',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware', # add this
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

CORS_ALLOW_ALL_ORIGINS = True # add this
```

## Deploy to Heroku
