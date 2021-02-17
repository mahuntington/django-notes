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

Let's test that the migrations worked.  There's a django terminal shell that will let us play around with the project without having to use the browser.  Let's start it:

```
python manage.py shell
```

Once it's started, we can write python to play around with the models.  In the shell run:

```python
from contacts_api.models import Contact
Contact.objects.all() # get all the contacts in the db
c = Contact(name="Matt", age=40) # create a new contact.  Note this isn't yet in the db
c.save() # save the contact to the db
c.id # check the id to make sure it's in the db
Contact.objects.all()
quit()
```

Django has a really nice admin app that lets us interface with the database from the browser.  In the terminal run the following and follow the prompts

```
python manage.py createsuperuser
```

Now add the following to contacts_api/admin.py

```python
from .models import Contact
admin.site.register(Contact)
```

and in the terminal run

```
python manage.py runserver
```

Go to http://localhost:8000/admin/ in the browser and sign in with the credentials you created when running `python manage.py createsuperuser`

## Create api endpoints

Now let's start working on the public facing API.  We'll use Django Rest Framework, which makes this job a little easier.

Install `djangorestframework`:

```
python -m pip install djangorestframework
```

edit django_rest_api/settings.py to tell django to use `djangorestframework`

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

Now we want to create a serializer for our Contact model.  This will take the data in our database and convert it to JSON.

Create contacts_api/serializers.py and add

```python
from rest_framework import serializers 
from .models import Contact 

class ContactSerializer(serializers.HyperlinkedModelSerializer): # serializers.HyperlinkedModelSerializer just tells django to convert sql to JSON
    class Meta:
        model = Contact # tell django which model to use
        fields = ('id', 'name', 'age',) # tell django which fields to include
```

Don't get thrown off by the nested class (`Meta`).  This is just an organizational thing that python lets us do.

Now lets create views which will connect the `ContactSerializer` with the `Contact` model.

- `generics.ListCreateAPIView` will be inherited by `ContactList` so that it will either display all Contacts in the DB or create a new one, depending on the request url and method
- `generics.RetrieveUpdateDestroyAPIView` will be inherited by `ContactDetail` so that it will either update or delete a contact in the DB, depending on the request url and method

set contacts_api/views.py to

```python
from rest_framework import generics
from .serializers import ContactSerializer
from .models import Contact

class ContactList(generics.ListCreateAPIView):
    queryset = Contact.objects.all() # tell django how to retrieve all objects from the DB
    serializer_class = ContactSerializer # tell django what serializer to use

class ContactDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Contact.objects.all()
    serializer_class = ContactSerializer
```

Now lets map request urls to the views we just created

create contacts_api/urls.py and add

```python
from django.urls import path
from . import views

urlpatterns = [
    path('api/contacts', views.ContactList.as_view(), name='contact_list'), # api/contacts will be routed to the ContactList view for handling
    path('api/contacts/<int:pk>', views.ContactDetail.as_view(), name='contact_detail'), # api/contacts will be routed to the ContactDetail view for handling
]
```

Finally register our contacts_api urls with django

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

## Add CORS

Lastly, let's allow web apps on other origins to access this api.  In the terminal, install the `django-cors-headers` package:

```
python -m pip install django-cors-headers
```

edit django_rest_api/settings.py to include the new package:

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
    'corsheaders.middleware.CorsMiddleware', # this makes the cors package run for all requests.  A bit like app.use() in express
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

### In Terminal

1. Create a heroku app from the root of your project folder, run: `heroku create` 

    - The above command will randomly generate a name for you, if you want to name your app something specific run: `heroku create urlNameYouWantHere`

1. Copy the heroku url that was created (without the `https://`), go to your `django_rest_api/settings.py` and add it into the `ALLOWED_HOSTS`

    - e.g. 

    ![](https://imgur.com/AVlB8kK.png)
    
### On the Browser 

1. Go to your heroku dashboard for the heroku project you just created
1. Click on Configure Add-Ons
1. Search for Heroku Postgres and add it

### In Terminal

1. run `heroku config:set DISABLE_COLLECTSTATIC=1`
1. `git add -A`
1. `git commit -m "heroku deployment"`
1. `git push heroku master` 
1. Once it builds successfully, run `heroku run bash` 
1. While in heroku bash,  apply the migrations to the heroku project by running: `python manage.py migrate` 
1. Still in heroku bash, create a superuser for the heroku project by running `python manage.py createsuperuser` and follow the prompts
    - To exit heroku bash, run `exit`

### In Browser 

1. After the migrations finish, you should now be able to open the heroku app in your browser to see the Django REST interface!
    - Don't forget to go to `/api/contacts`
1. Remember that your heroku database is separate from your local database, so there should not be any data on the first load. 
    - You can add data by logging in with the heroku superuser you created
1. You can now use this deployed version as your backend API
